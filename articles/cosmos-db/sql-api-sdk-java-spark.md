---
title: 适用于 SQL API 的 Azure Cosmos DB Apache Spark 连接器发行说明和资源
description: 了解有关适用于 SQL API 的 Azure Cosmos DB Apache Spark 连接器的所有信息，包括发行日期、停用日期和 Azure Cosmos DB SQL Async Java SDK 各版本之间所做的更改。
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
origin.date: 08/12/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.custom: devx-track-java
ms.openlocfilehash: 18ab0618a0497247a12d03f15ad9113275cc7d76
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246835"
---
<!--Verified successfully-->
# <a name="azure-cosmos-db-apache-spark-connector-for-core-sql-api-release-notes-and-resources"></a>适用于核心 (SQL) API 的 Azure Cosmos DB Apache Spark 连接器：发行说明和资源
> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [.NET SDK v2](sql-api-sdk-dotnet.md)
> * [.NET Core SDK v2](sql-api-sdk-dotnet-core.md)
> * [.NET 更改源 SDK v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [Java SDK v4](sql-api-sdk-java-v4.md)
> * [Async Java SDK v2](sql-api-sdk-async-java.md)
> * [Sync Java SDK v2](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Spark 连接器](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](https://docs.microsoft.com/rest/api/cosmos-db/)
> * [REST 资源提供程序](https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [批量执行工具 - .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [批量执行程序 - Java](sql-api-sdk-bulk-executor-java.md)

通过使用适用于核心 (SQL) 的 Azure Cosmos DB Apache Spark 连接器加速大数据分析。 利用 Spark 连接器，可以对 Azure Cosmos DB 中存储的数据运行 [Spark](https://spark.apache.org/) 作业。 支持批处理和流处理。

可以结合 [Azure HDInsight](https://www.azure.cn/home/features/hdinsight/)（在 Azure 上提供托管的 Spark 群集）使用连接器。 下表显示了支持的 Spark 版本。

<!--Not Available on [Azure Databricks](https://www.azure.cn/home/features/databricks/)-->

| 组件 | 版本 |
|---------|-------|
| Apache Spark | 2.4.x、2.3.x、2.2.x 和 2.1.x |
| Scala | 2.11 |
| Azure Databricks 运行时版本 | > 3.4 |

> [!WARNING]
> 此连接器支持 Azure Cosmos DB 的核心 (SQL) API。
> 对于 Cosmos DB for MongoDB API，请使用 [MongoDB Spark 连接器](https://docs.mongodb.com/spark-connector/master/)。
> 对于 Cosmos DB Cassandra API，请使用 [Cassandra Spark 连接器](https://github.com/datastax/spark-cassandra-connector)。
>

## <a name="helpful-content"></a>有用的内容

| Content | 链接 |
|---|---|
| **SDK 下载** | [从 Apache Spark 下载](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG) |
|**API 文档** | [Spark 连接器参考]() |
|**参与 SDK** | [GitHub 上适用于 Apache Spark 的 Azure Cosmos DB 连接器](https://github.com/Azure/azure-cosmosdb-spark) | 
|**入门** | [使用 Apache Spark 到 Azure Cosmos DB 的连接器加速大数据分析](/cosmos-db/spark-connector#bk_working_with_connector) <br /> [将 Apache Spark 结构化流式处理与 Apache Kafka 和 Azure Cosmos DB 配合使用](/hdinsight/apache-kafka-spark-structured-streaming-cosmosdb?toc=/cosmos-db/toc.json&bc=/cosmos-db/breadcrumb/toc.json) | 

## <a name="release-history"></a>版本历史记录

### <a name="311"></a>3.1.1
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了流式处理检查点边缘案例，其中“id”内包含“|”字符，并且应用了“ChangeFeedMaxPagesPerBatch”配置

### <a name="310"></a>3.1.0
#### <a name="new-features"></a>新增功能
* 添加了在使用嵌套分区键时对批量更新的支持
* 添加了在写入 Cosmos DB 过程中对 Decimal 和 Float 数据类型的支持。
* 添加了对 Timestamp 类型使用 Long (unix Epoch) 作为值时对这些类型的支持
#### <a name="key-bug-fixes"></a>关键 bug 修复

### <a name="308"></a>3.0.8
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了使用“WriteThroughputBudget”配置时的类型强制转换异常。

### <a name="307"></a>3.0.7
#### <a name="new-features"></a>新增功能
* 将批量失败的错误信息添加到异常和日志。
#### <a name="key-bug-fixes"></a>关键 bug 修复

### <a name="306"></a>3.0.6
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了流式处理检查点问题。

### <a name="305"></a>3.0.5
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了无意中留下的“错误”级别消息的日志级别，以减少干扰

### <a name="304"></a>3.0.4
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了分区拆分过程中结构化流式处理中的 bug，该 bug 可能导致缺少某些更改源记录或在执行检查点写入操作时出现 Null 异常

### <a name="303"></a>3.0.3
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了为 readStream 提供的自定义架构被忽略的 bug

### <a name="302"></a>3.0.2
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了回归（未改装的 JAR 包括所有已改装的依赖项），该 bug 导致生成时间增加了 50%

### <a name="301"></a>3.0.1
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 bug 修复
* 修复了一个依赖项问题，该问题导致通过 TCP 的直接传输失败，出现 RequestTimeoutException

### <a name="300"></a>3.0.0
#### <a name="new-features"></a>新增功能
* 改进了连接管理和连接池以减少元数据调用数
#### <a name="key-bug-fixes"></a>关键 bug 修复

## <a name="faq"></a>常见问题
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>后续步骤

若要了解有关 Cosmos DB 的详细信息，请参阅 [Azure Cosmos DB](https://www.azure.cn/home/features/cosmos-db/) 服务页。

若要详细了解 Apache Spark，请参阅[主页](https://spark.apache.org/)。

<!-- Update_Description: update meta properties, wording update, update link -->