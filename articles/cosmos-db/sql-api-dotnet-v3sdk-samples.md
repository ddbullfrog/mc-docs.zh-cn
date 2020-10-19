---
title: Azure Cosmos DB - SQL API 的 .NET (Microsoft.Azure.Cosmos) 示例
description: 在 GitHub 上查找使用 Azure Cosmos DB SQL API 的常见任务的 C# .NET V3 SDK 示例。
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
origin.date: 10/07/2019
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: devx-track-dotnet
ms.openlocfilehash: 2d0637a008b3b2e7649664593d2d069baa1c0feb
ms.sourcegitcommit: 7320277f4d3c63c0b1ae31ba047e31bf2fe26bc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92118012"
---
# <a name="azure-cosmos-dbnet-v3-sdk-microsoftazurecosmos-examples-for-the-sql-api"></a>SQL API 的 Azure Cosmos DB.NET V3 SDK (Microsoft.Azure.Cosmos) 示例

> [!div class="op_single_selector"]
> * [.NET V2 SDK 示例](sql-api-dotnet-samples.md)
> * [.NET V3 SDK 示例](sql-api-dotnet-v3sdk-samples.md)
> * [Java V4 SDK 示例](sql-api-java-sdk-samples.md)
> * [Spring Data V3 SDK 示例](sql-api-spring-data-sdk-samples.md)
> * [Node.js 示例](sql-api-nodejs-samples.md)
> * [Python 示例](sql-api-python-samples.md)
> * [Azure 代码示例库](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
>
>

[azure-cosmos-dotnet-v3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage) GitHub 存储库中包含可对 Azure Cosmos DB 资源执行 CRUD 操作和其他常见操作的最新 .NET 示例解决方案。 如果你熟悉旧版 .NET SDK，则可能习惯使用术语“集合”和“文档”。 由于 Azure Cosmos DB 支持多个 API 模型，因此 3.0 版的 .NET SDK 使用通用术语“容器”和“项”。 容器可以是集合、图或表。 项可以是文档、边缘/顶点或行，是容器中的内容。 本文将提供：

* 指向每个示例 C# 项目文件中各项任务的链接。
* 指向相关的 API 参考内容的链接。

## <a name="prerequisites"></a>先决条件

已安装包含 Azure 开发工作流的 Visual Studio 2019

- 你可以下载并使用**免费的** [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。 

    [Microsoft.Azure.cosmos NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.cosmos/)

Azure 订阅，或免费的 Cosmos DB 试用帐户

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

- 可以[激活 Visual Studio 订阅者权益](https://www.azure.cn/offers/ms-mc-arz-msdn//)：Visual Studio 订阅每月提供可用来试用付费版 Azure 服务的信用额度。
- [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]  

> [!NOTE]
> 这些示例都是独立的，运行完成后，可对其进行设置和清理。 每次执行此操作时，都会根据容器的性能层，对订阅收取使用 1 小时的费用。
> 

## <a name="database-examples"></a>数据库示例

*DatabaseManagement* 项目示例的 [RunDatabaseDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L65-L91) 方法演示如何执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos 数据库，请参阅[使用数据库、容器和项](databases-containers-items.md)。

| 任务 | API 参考 |
| --- | --- |
| [创建数据库](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L68) |[CosmosClient.CreateDatabaseIfNotExistsAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync) |
| [按 ID 读取数据库](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L80) |[Database.ReadAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.readasync) |
| [读取帐户的所有数据库](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L96) |[CosmosClient.GetDatabaseQueryIterator](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getdatabasequeryiterator) |
| [删除数据库](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/DatabaseManagement/Program.cs#L106) |[Database.DeleteAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.deleteasync) |

## <a name="container-examples"></a>容器示例

*ContainerManagement* 项目示例的 [RunContainerDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L69-L89) 方法演示如何执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos 容器，请参阅[使用数据库、容器和项](databases-containers-items.md)。

| 任务 | API 参考 |
| --- | --- |
| [创建容器](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L97-L107) |[Database.CreateContainerIfNotExistsAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync) |
| [使用自定义索引策略创建容器](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L111-L127) |[Database.CreateContainerIfNotExistsAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync) |
| [更改容器的已配置性能](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L149-L171) |[Container.ReplaceThroughputAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.replacethroughputasync) |
| [按 ID 获取容器](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L176-L185) |[Container.ReadContainerAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readcontainerasync) |
| [读取数据库中的所有容器](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L193-L205) |[Database.GetContainerQueryIterator](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.getcontainerqueryiterator) |
| [删除容器](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ContainerManagement/Program.cs#L213-L2018) |[Container.DeleteContainerAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deletecontainerasync) |

## <a name="item-examples"></a>项示例

*ItemManagement* 项目示例的 [RunItemsDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L119-L130) 方法演示如何执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos 项，请参阅[使用数据库、容器和项](databases-containers-items.md)。

| 任务 | API 参考 |
| --- | --- |
| [创建项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L161-L200) |[Container.CreateItemAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync) |
| [按 ID 读取项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L203-L241) |[container.ReadItemAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync) |
| [查询项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L244-L320) |[container.GetItemQueryIterator](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator) |
| [替换项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L323-L363) |[container.ReplaceItemAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.replaceitemasync) |
| [更新插入项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L366-L411) |[container.UpsertItemAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync) |
| [删除项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L414-L424) |[container.DeleteItemAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync) |
| [使用条件 ETag 检查替换项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L585-L674) |[RequestOptions.IfMatchEtag](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.requestoptions.ifmatchetag) |

## <a name="indexing-examples"></a>索引示例

*IndexManagement* 项目示例的 [RunIndexDemo](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L108-L122) 方法演示如何执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos DB 中的索引，请参阅[索引策略](index-policy.md)、[索引类型](index-types.md)和[索引路径](index-paths.md)。 

| 任务 | API 参考 |
| --- | --- |
| [从索引中排除项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L130-L186) |[IndexingDirective.Exclude](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingdirective) |
| [使用延迟索引](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L198-L220) |[IndexingPolicy.IndexingMode](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode) |
| [从索引中排除指定的项路径](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/IndexManagement/Program.cs#L230-L297) |[IndexingPolicy.ExcludedPaths](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths) |

## <a name="query-examples"></a>查询示例

示例 *Queries* 项目的 [RunDemoAsync](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L76-L96) 方法显示了如何使用 SQL 查询语法、查询的 LINQ 提供程序和 Lambda 执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos DB 中的 SQL 查询引用，请参阅 [Azure Cosmos DB 的 SQL 查询示例](how-to-sql-query.md)。

| 任务 | API 参考 |
| --- | --- |
| [查询单个分区中的项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L154-L186) |[container.GetItemQueryIterator](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator) |
| [查询多个分区中的项](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L215-L275) |[container.GetItemQueryIterator](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator) |
| [使用 SQL 语句进行查询](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L189-L212) |[container.GetItemQueryIterator](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator) |

## <a name="change-feed-examples"></a>更改源示例

示例 *ChangeFeed* 项目的 [RunBasicChangeFeed](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L91-L119) 方法演示如何执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos DB 中的更改源，请参阅[读取 Azure Cosmos DB 更改源](read-change-feed.md)和[更改源处理器](change-feed-processor.md)。

| 任务 | API 参考 |
| --- | --- |
| [基本的更改源功能](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L91-L119) |[Container.GetChangeFeedProcessorBuilder](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder) |
| [读取特定时间的更改源](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L127-L162) |[Container.GetChangeFeedProcessorBuilder](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder) |
| [从头读取更改源](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L170-L198) |[ChangeFeedProcessorBuilder.WithStartTime(DateTime)](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withstarttime) |
| [从更改源处理器迁移到 V3 SDK 中的更改源](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs#L256-L333) |[Container.GetChangeFeedProcessorBuilder](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder) |

## <a name="server-side-programming-examples"></a>服务器端编程示例

示例 *ServerSideScripts* 项目的 [RunDemoAsync](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L72-L102) 方法演示如何执行以下任务。 若要在运行以下示例之前了解 Azure Cosmos DB 中的服务器端编程，请参阅[存储过程、触发器和用户定义的函数](stored-procedures-triggers-udfs.md)。

| 任务 | API 参考 |
| --- | --- |
| [创建存储过程](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L116) |[Scripts.CreateStoredProcedureAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.scripts.createstoredprocedureasync) |
| [执行存储过程](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L135) |[Scripts.ExecuteStoredProcedureAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.scripts.executestoredprocedureasync) |
| [删除存储过程](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ServerSideScripts/Program.cs#L351) |[Scripts.DeleteStoredProcedureAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.scripts.deletestoredprocedureasync) |

<!-- Update_Description: update meta properties, wording update, update link -->