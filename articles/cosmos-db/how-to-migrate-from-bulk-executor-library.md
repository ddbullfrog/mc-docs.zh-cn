---
title: 从批量执行工具库迁移到 Azure Cosmos DB .NET V3 SDK 中的批量操作支持
description: 了解如何将应用程序从使用批量执行工具库迁移到使用 Azure Cosmos DB SDK V3 中的批量操作支持
ms.service: cosmos-db
ms.topic: how-to
origin.date: 04/24/2020
author: rockboyfor
ms.date: 08/17/2020
ms.testscope: yes
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.custom: devx-track-dotnet
ms.openlocfilehash: 2e3a42dd1654fcb6fe7601443ee9653b7b4772af
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246438"
---
<!--Verified with cosmos db SDK V3-->
# <a name="migrate-from-the-bulk-executor-library-to-the-bulk-support-in-azure-cosmos-db-net-v3-sdk"></a>从批量执行工具库迁移到 Azure Cosmos DB .NET V3 SDK 中的批量操作支持

本文介绍需要执行哪些步骤，才能将使用 [.NET 批量执行工具库](bulk-executor-dot-net.md)的现有应用程序的代码迁移到使用最新版 .NET SDK 中的[批量操作支持](tutorial-sql-api-dotnet-bulk-import.md)功能。

## <a name="enable-bulk-support"></a>启用批量操作支持

通过 [AllowBulkExecution](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.allowbulkexecution) 配置在 `CosmosClient` 实例上启用批量操作支持：

   ```csharp
   new CosmosClient(endpoint, authKey, new CosmosClientOptions() { AllowBulkExecution = true });

   ```

## <a name="create-tasks-for-each-operation"></a>为每个操作创建任务

通过利用[任务并行库](https://docs.microsoft.com//dotnet/standard/parallel-programming/task-parallel-library-tpl)，并将并发执行的操作分组，.NET SDK 中的批量操作支持得以发挥作用。 

SDK 中没有任何一种方法可将文档或操作的列表用作输入参数，你需要针对要批量执行的每个操作创建一个任务，然后等待它们完成。

例如，如果初始输入是一个项列表，其中的每个项采用以下架构：

   ```csharp
   public class MyItem
   {
       public string id { get; set; }

       public string pk { get; set; }

       public int operationCounter { get; set; } = 0;
   }

   ```

若要执行批量导入（类似于使用 BulkExecutor.BulkImportAsync），需要对 `CreateItemAsync` 发出并发调用。 例如：

   ```csharp
   BulkOperations<MyItem> bulkOperations = new BulkOperations<MyItem>(documentsToWorkWith.Count);
   foreach (MyItem document in documentsToWorkWith)
   {
       bulkOperations.Tasks.Add(container.CreateItemAsync(document, new PartitionKey(document.pk)).CaptureOperationResponse(document));
   }

   ```

若要执行批量更新（类似于使用 [BulkExecutor.BulkUpdateAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkupdateasync)），则在更新项值后，需要对 `ReplaceItemAsync` 方法发出并发调用。 例如：

   ```csharp
   BulkOperations<MyItem> bulkOperations = new BulkOperations<MyItem>(documentsToWorkWith.Count);
   foreach (MyItem document in documentsToWorkWith)
   {
       document.operationCounter++;
       bulkOperations.Tasks.Add(container.ReplaceItemAsync(document, document.id, new PartitionKey(document.pk)).CaptureOperationResponse(document));
   }

   ```

若要执行批量删除（类似于使用 [BulkExecutor.BulkDeleteAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkdeleteasync)），需要使用每个项的 `id` 和分区键对 `DeleteItemAsync` 发出并发调用。 例如：

   ```csharp
   BulkOperations<MyItem> bulkOperations = new BulkOperations<MyItem>(documentsToWorkWith.Count);
   foreach (MyItem document in documentsToWorkWith)
   {
       document.operationCounter++;
       bulkOperations.Tasks.Add(container.DeleteItemAsync<MyItem>(document.id, new PartitionKey(document.pk)).CaptureOperationResponse(document));
   }

   ```

## <a name="capture-task-result-state"></a>捕获任务结果状态

在前面的代码示例中，我们已创建一个并发任务列表，并对其中的每个任务调用了 `CaptureOperationResponse` 方法。 此方法是一个扩展，可让我们通过捕获任何错误并跟踪[请求单位用量](request-units.md)来保持与 BulkExecutor 类似的响应架构。

   ```csharp
   public static Task<OperationResponse<T>> CaptureOperationResponse<T>(this Task<ItemResponse<T>> task, T item)
   {
       return task.ContinueWith(itemResponse =>
       {
           if (itemResponse.IsCompletedSuccessfully)
           {
               return new OperationResponse<T>()
               {
                   Item = item,
                   IsSuccessful = true,
                   RequestUnitsConsumed = task.Result.RequestCharge
               };
           }

           AggregateException innerExceptions = itemResponse.Exception.Flatten();
           if (innerExceptions.InnerExceptions.FirstOrDefault(innerEx => innerEx is CosmosException) is CosmosException cosmosException)
           {
               return new OperationResponse<T>()
               {
                   Item = item,
                   RequestUnitsConsumed = cosmosException.RequestCharge,
                   IsSuccessful = false,
                   CosmosException = cosmosException
               };
           }

           return new OperationResponse<T>()
           {
               Item = item,
               IsSuccessful = false,
               CosmosException = innerExceptions.InnerExceptions.FirstOrDefault()
           };
       });
   }

   ```

其中，`OperationResponse` 声明为：

   ```csharp
   public class OperationResponse<T>
   {
       public T Item { get; set; }
       public double RequestUnitsConsumed { get; set; } = 0;
       public bool IsSuccessful { get; set; }
       public Exception CosmosException { get; set; }
   }

   ```

## <a name="execute-operations-concurrently"></a>并发执行操作

为了跟踪整个 Tasks 列表的作用域，我们使用以下帮助程序类：

   ```csharp
   public class BulkOperations<T>
   {
       public readonly List<Task<OperationResponse<T>>> Tasks;

       private readonly Stopwatch stopwatch = Stopwatch.StartNew();

       public BulkOperations(int operationCount)
       {
           this.Tasks = new List<Task<OperationResponse<T>>>(operationCount);
       }

       public async Task<BulkOperationResponse<T>> ExecuteAsync()
       {
           await Task.WhenAll(this.Tasks);
           this.stopwatch.Stop();
           return new BulkOperationResponse<T>()
           {
               TotalTimeTaken = this.stopwatch.Elapsed,
               TotalRequestUnitsConsumed = this.Tasks.Sum(task => task.Result.RequestUnitsConsumed),
               SuccessfulDocuments = this.Tasks.Count(task => task.Result.IsSuccessful),
               Failures = this.Tasks.Where(task => !task.Result.IsSuccessful).Select(task => (task.Result.Item, task.Result.CosmosException)).ToList()
           };
       }
   }

   ```

`ExecuteAsync` 方法会等待所有操作完成，你可以像这样使用它：

   ```csharp
   BulkOperationResponse<MyItem> bulkOperationResponse = await bulkOperations.ExecuteAsync();

   ```

## <a name="capture-statistics"></a>捕获统计信息

以上代码将等到所有操作完成，然后计算所需的统计信息。 这些统计信息类似于批量执行工具库的 [BulkImportResponse](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkimport.bulkimportresponse) 的统计信息。

   ```csharp
   public class BulkOperationResponse<T>
   {
       public TimeSpan TotalTimeTaken { get; set; }
       public int SuccessfulDocuments { get; set; } = 0;
       public double TotalRequestUnitsConsumed { get; set; } = 0;

       public IReadOnlyList<(T, Exception)> Failures { get; set; }
   }

   ```

`BulkOperationResponse` 包含：

1. 通过批量操作支持处理操作列表所用的总时间。
1. 成功的操作数目。
1. 消耗的请求单位总数。
1. 如果发生失败，它会显示一个元组列表，其中包含了异常和关联的项，以用于日志记录和问题识别目的。

## <a name="retry-configuration"></a>重试配置

批量执行工具库提供了[指导](bulk-executor-dot-net.md#bulk-import-data-to-an-azure-cosmos-account)，其中指出，需要将 [RetryOptions](https://docs.azure.cn/dotnet/api/microsoft.azure.documents.client.connectionpolicy.retryoptions) 的 `MaxRetryWaitTimeInSeconds` 和 `MaxRetryAttemptsOnThrottledRequests` 设置为 `0`，以将控制权委托给该库。

对于 .NET SDK 中的批量操作支持，不存在隐藏的行为。 可以直接通过 [CosmosClientOptions.MaxRetryAttemptsOnRateLimitedRequests](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.maxretryattemptsonratelimitedrequests) 和 [CosmosClientOptions.MaxRetryWaitTimeOnRateLimitedRequests](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.maxretrywaittimeonratelimitedrequests) 配置重试选项。

> [!NOTE]
> 如果从数据量来看，预配的请求单位数远远低于预期数量，则你可能需要考虑将这些参数设置为较高的值。 批量操作会花费更长时间，但由于重试次数较高，该操作成功完成的可能性较高。

## <a name="performance-improvements"></a>性能改进

与 .NET SDK 中的其他操作一样，使用流 API 可以提高性能，并避免任何不必要的序列化。 

仅当所用数据的性质与字节流（例如文件流）的性质匹配时，才可以使用流 API。 在这种情况下，使用 `CreateItemStreamAsync`、`ReplaceItemStreamAsync` 或 `DeleteItemStreamAsync` 方法并使用 `ResponseMessage`（而不是 `ItemResponse`）可以增加可实现的吞吐量。

## <a name="next-steps"></a>后续步骤

* 若要详细了解 .NET SDK 发行版，请参阅 [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) 一文。
* 从 GitHub 获取完整的[迁移源代码](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/BulkExecutorMigration)。
* [GitHub 上的其他批量操作示例](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/BulkSupport)

<!-- Update_Description: update meta properties, wording update, update link -->