---
title: 使用 Azure 数据资源管理器 Java SDK 引入数据
description: 本文介绍如何使用 Java SDK 将数据引入（加载）到 Azure 数据资源管理器中。
author: orspod
ms.author: v-tawe
ms.reviewer: abhishgu
ms.service: data-explorer
ms.topic: how-to
origin.date: 10/22/2020
ms.date: 10/30/2020
ms.openlocfilehash: 2fb11c4cd4ebc57677f17b79d7a8727215f8750f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106533"
---
# <a name="ingest-data-using-the-azure-data-explorer-java-sdk"></a>使用 Azure 数据资源管理器 Java SDK 引入数据 

> [!div class="op_single_selector"]
> * [.NET](net-sdk-ingest-data.md)
> * [Python](python-ingest-data.md)
> * [Node](node-ingest-data.md)
> * [Go](go-ingest-data.md)
> * [Java](java-ingest-data.md)

Azure 数据资源管理器是一项快速且高度可缩放的数据探索服务，适用于日志和遥测数据。 [Java 客户端库](kusto/api/java/kusto-java-client-library.md)可用于引入数据、发出控制命令和在 Azure 数据资源管理器群集中查询数据。

本文介绍如何使用 Azure 数据资源管理器 Java 库引入数据。 首先，你将在测试群集中创建一个表和数据映射。 然后你将使用 Java SDK 从 blob 存储中将“引入”排入群集的队列中并验证结果。

## <a name="prerequisites"></a>先决条件

* [试用帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。
* JDK 版本 1.8 或更高版本。
* [Maven](https://maven.apache.org/download.cgi)。
* 一个[群集和数据库](create-cluster-database-portal.md)。
<!-- * Create an [App Registration and grant it permissions to the database](provision-azure-ad-app.md). Save the client ID and client secret for later use. -->

## <a name="review-the-code"></a>查看代码

本部分是可选的。 查看以下代码片段，了解代码的工作原理。 若要跳过此部分，请转到[运行应用程序](#run-the-application)。

### <a name="authentication"></a>身份验证

该程序将 Azure Active Directory 身份验证凭据与 ConnectionStringBuilder` 结合使用。

1. 创建用于查询和管理的 `com.microsoft.azure.kusto.data.Client`。

    ```java
    static Client getClient() throws Exception {
        ConnectionStringBuilder csb = ConnectionStringBuilder.createWithAadApplicationCredentials(endpoint, clientID, clientSecret, tenantID);
        return ClientFactory.createClient(csb);
    }
    ```

1. 创建并使用 `com.microsoft.azure.kusto.ingest.IngestClient` 将“数据引入”排入 Azure 数据资源管理器中的队列：

    ```java
    static IngestClient getIngestionClient() throws Exception {
        String ingestionEndpoint = "https://ingest-" + URI.create(endpoint).getHost();
        ConnectionStringBuilder csb = ConnectionStringBuilder.createWithAadApplicationCredentials(ingestionEndpoint, clientID, clientSecret);
        return IngestClientFactory.createClient(csb);
    }
    ```

### <a name="management-commands"></a>管理命令

通过调用 `com.microsoft.azure.kusto.data.Client` 对象上的 `execute` 来执行[控制命令](kusto/management/commands.md)（如 [`.drop`](kusto/management/drop-function.md) 和 [`.create`](kusto/management/create-function.md)）。

例如，按如下所示创建 `StormEvents` 表：

```java
static final String createTableCommand = ".create table StormEvents (StartTime: datetime, EndTime: datetime, EpisodeId: int, EventId: int, State: string, EventType: string, InjuriesDirect: int, InjuriesIndirect: int, DeathsDirect: int, DeathsIndirect: int, DamageProperty: int, DamageCrops: int, Source: string, BeginLocation: string, EndLocation: string, BeginLat: real, BeginLon: real, EndLat: real, EndLon: real, EpisodeNarrative: string, EventNarrative: string, StormSummary: dynamic)";

static void createTable(String database) {
    try {
        getClient().execute(database, createTableCommand);
        System.out.println("Table created");
    } catch (Exception e) {
        System.out.println("Failed to create table: " + e.getMessage());
        return;
    }
    
}
```

### <a name="data-ingestion"></a>数据引入

使用现有 Azure Blob 存储容器中的文件将“引入”排入队列。 
* 使用 `BlobSourceInfo` 来指定 Blob 存储路径。
* 使用 `IngestionProperties` 定义表、数据库、映射名称和数据类型。 在以下示例中，数据类型是 `CSV`。

```java
    ...
    static final String blobPathFormat = "https://%s.blob.core.chinacloudapi.cn/%s/%s%s";
    static final String blobStorageAccountName = "kustosamplefiles";
    static final String blobStorageContainer = "samplefiles";
    static final String fileName = "StormEvents.csv";
    static final String blobStorageToken = "??st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    ....

    static void ingestFile(String database) throws InterruptedException {
        String blobPath = String.format(blobPathFormat, blobStorageAccountName, blobStorageContainer,
                fileName, blobStorageToken);
        BlobSourceInfo blobSourceInfo = new BlobSourceInfo(blobPath);

        IngestionProperties ingestionProperties = new IngestionProperties(database, tableName);
        ingestionProperties.setDataFormat(DATA_FORMAT.csv);
        ingestionProperties.setIngestionMapping(ingestionMappingRefName, IngestionMappingKind.Csv);
        ingestionProperties.setReportLevel(IngestionReportLevel.FailuresAndSuccesses);
        ingestionProperties.setReportMethod(IngestionReportMethod.QueueAndTable);
    ....
```

引入过程在单独的线程中开始，`main` 线程等待引入线程完成。 此过程使用 [CountdownLatch](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CountDownLatch.html)。 引入 API (`IngestClient#ingestFromBlob`) 不是异步的。 `while` 循环每 5 秒轮询一次当前状态，并等待引入状态从 `Pending` 更改为不同状态。 最终状态可以是 `Succeeded`、`Failed` 或 `PartiallySucceeded`。

```java
        ....
        CountDownLatch ingestionLatch = new CountDownLatch(1);
        new Thread(new Runnable() {
            @Override
            public void run() {
                IngestionResult result = null;
                try {
                    result = getIngestionClient().ingestFromBlob(blobSourceInfo, ingestionProperties);
                } catch (Exception e) {
                    ingestionLatch.countDown();
                }
                try {
                    IngestionStatus status = result.getIngestionStatusCollection().get(0);
                    while (status.status == OperationStatus.Pending) {
                        Thread.sleep(5000);
                        status = result.getIngestionStatusCollection().get(0);
                    }
                    ingestionLatch.countDown();
                } catch (Exception e) {
                    ingestionLatch.countDown();
                }
            }
        }).start();
        ingestionLatch.await();
    }
```

> [!TIP]
> 还有其他为不同应用程序异步处理引入的方法。 例如，可以使用 [`CompletableFuture`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) 来创建定义引入后操作的管道（如查询表），或处理向 `IngestionStatus` 报告的异常。

## <a name="run-the-application"></a>运行此应用程序

### <a name="general"></a>常规

运行示例代码时，将执行以下操作：

   1. **删除表** ：删除 `StormEvents` 表（如果存在）。
   1. **创建表** ：创建 `StormEvents` 表。
   1. **创建映射** ：创建 `StormEvents_CSV_Mapping` 映射。
   1. **引入文件** ：一个 CSV 文件（在 Azure Blob 存储中）将排队等待引入。

以下为示例代码来自 `App.java`：

```java
public static void main(final String[] args) throws Exception {
    dropTable(database);
    createTable(database);
    createMapping(database);
    ingestFile(database);
}
```

> [!TIP]
> 若要尝试不同的操作组合，请在 `App.java` 中取消注释/注释相应的方法。

### <a name="run-the-application"></a>运行此应用程序

1. 从 GitHub 克隆示例代码：

    ```console
    git clone https://github.com/Azure-Samples/azure-data-explorer-java-sdk-ingest.git
    cd azure-data-explorer-java-sdk-ingest
    ```

1. 使用以下信息作为程序使用的环境变量来设置服务主体信息：
    * 群集终结点 
    * 数据库名称 

    ```console
    export AZURE_SP_CLIENT_ID="<replace with appID>"
    export AZURE_SP_CLIENT_SECRET="<replace with password>"
    export KUSTO_ENDPOINT="https://<cluster name>.<azure region>.kusto.chinacloudapi.cn"
    export KUSTO_DB="name of the database"
    ```

1. 生成并运行：

    ```console
    mvn clean package
    java -jar target/adx-java-ingest-jar-with-dependencies.jar
    ```

    输出结果会类似于：

    ```console
    Table dropped
    Table created
    Mapping created
    Waiting for ingestion to complete...
    ```

等待几分钟完成引入过程。 成功完成后，你将看到以下日志消息：`Ingestion completed successfully`。 此时，你可以退出程序，并转到下一步，这不会影响已排队的引入进程。

## <a name="validate"></a>验证

等待 5 到 10 分钟，以便已排队的引入调度引入进程并将数据加载到 Azure 数据资源管理器中。 

1. 登录到 [https://dataexplorer.azure.com](https://dataexplorer.azure.cn) 并连接到群集。 
1. 运行以下命令，以获取 `StormEvents` 表中的记录的计数：
    
    ```kusto
    StormEvents | count
    ```

## <a name="troubleshoot"></a>故障排除

1. 若要查看过去四个小时内是否存在任何失败引入，请在数据库中运行以下命令： 

    ```kusto
    .show ingestion failures
    | where FailedOn > ago(4h) and Database == "<DatabaseName>"
    ```

1. 若要查看过去四个小时内所有引入操作的状态，请运行以下命令：

    ```kusto
    .show operations
    | where StartedOn > ago(4h) and Database == "<DatabaseName>" and Operation == "DataIngestPull"
    | summarize arg_max(LastUpdatedOn, *) by OperationId
    ```

## <a name="clean-up-resources"></a>清理资源

如果不打算使用刚刚创建的资源，请在数据库中运行以下命令，删除 `StormEvents` 表。

```kusto
.drop table StormEvents
```

## <a name="next-steps"></a>后续步骤

[编写查询](write-queries.md)
