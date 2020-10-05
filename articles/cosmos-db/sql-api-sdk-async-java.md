---
title: Azure Cosmos DB - SQL Async Java API、SDK 和资源
description: 了解有关 SQL Async Java API 和 SDK 的所有信息，包括发布日期、停用日期和 Azure Cosmos DB SQL Async Java SDK 各版本之间所做的更改。
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
origin.date: 08/12/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: devx-track-java
ms.openlocfilehash: 1db880fae3c8f7bee60fc3c0d207c02b197e29fb
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246373"
---
# <a name="azure-cosmos-db-async-java-sdk-for-sql-api-release-notes-and-resources"></a>适用于 SQL API 的 Azure Cosmos DB Async Java SDK：发行说明和资源
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
> * [REST](https://docs.microsoft.com/rest/api
> * [REST 资源提供程序](/azure-resource-manager/management/azure-services-resource-providers)
> * [SQL](sql-api-query-reference.md)
> * [批量执行工具 - .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [批量执行程序 - Java](sql-api-sdk-bulk-executor-java.md)

SQL API Async Java SDK 与 SQL API Java SDK 的区别在于，前者通过支持 [Netty 库](https://netty.io/)提供异步操作。 先存在的 [SQL API Java SDK](sql-api-sdk-java.md) 不支持异步操作。 

> [!IMPORTANT]  
> 这不是最新的 Azure Cosmos DB Java SDK！ 请考虑将 [Azure Cosmos DB Java SDK v4](sql-api-sdk-java-v4.md) 用于项目。 若要升级，请按照[迁移到 Azure Cosmos DB Java SDK v4](migrate-java-v4-sdk.md) 指南和 [Reactor 与 RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md) 指南中的说明进行操作。 
>

| |  |
|---|---|
| **SDK 下载** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) |
|**API 文档** |[Java API 参考文档](https://docs.microsoft.com/java/api/com.microsoft.azure.cosmosdb.rx.asyncdocumentclient) | 
|**参与 SDK** | [GitHub](https://github.com/Azure/azure-cosmosdb-java) | 
|**入门** | [Async Java SDK 入门](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-async-java-getting-started) | 
|**代码示例** | [GitHub](https://github.com/Azure/azure-cosmosdb-java#usage-code-sample)| 
| **性能提示**| [GitHub 自述文件](https://github.com/Azure/azure-cosmosdb-java#guide-for-prod)| 
| 受支持的最小运行时|[JDK 8](https://docs.microsoft.com/java/azure/jdk/) | 

## <a name="2610---2020-08-31"></a>2.6.10 - 2020-08-31
- 解决了直接 tcp 调用的区域故障转移问题；将在其他可用区域中重试因网络而失败的直接 tcp 调用，但不会将当前区域标记为不可用，随后的 tcp 请求仍将发送到当前区域。([#351](https://github.com/Azure/azure-cosmosdb-java/pull/351))

## <a name="269---2020-06-15"></a>2.6.9 - 2020-06-15
- 修复了从继续标记恢复时 order by 结果重复的问题 ([#341](https://github.com/Azure/azure-cosmosdb-java/pull/341))

## <a name="268---2020-05-11"></a>2.6.8 - 2020-05-11
- 修复了一个问题，该问题导致从继续标记恢复时 order by 结果可能包含重复/缺少的文档 ([#332](https://github.com/Azure/azure-cosmosdb-java/pull/332))

## <a name="267---2020-04-21"></a>2.6.7 - 2020-04-21
- 修复了直接 TCP 传输客户端中的多个内存泄漏 ([#326](https://github.com/Azure/azure-cosmosdb-java/pull/326))

## <a name="266---2020-03-12"></a>2.6.6 - 2020-03-12
- 使 DocumentClientException 线程安全 ([#323](https://github.com/Azure/azure-cosmosdb-java/pull/323))

## <a name="265---2020-01-31"></a>2.6.5 - 2020-01-31
- 修复了尝试恢复跨分区查询时继续标记设置不正确的问题 ([#312](https://github.com/Azure/azure-cosmosdb-java/pull/312))
- 将反序列化移出 Netty 线程 ([#315](https://github.com/Azure/azure-cosmosdb-java/pull/315))
- 修复了禁止时无法重试写入的问题 ([#307](https://github.com/Azure/azure-cosmosdb-java/pull/307))

## <a name="264---2020-01-02"></a>2.6.4 - 2020-01-02
- 修复了 SDK 在发生网络故障时重试写入的 bug ([#301](https://github.com/Azure/azure-cosmosdb-java/pull/301))
- 添加了对使用系统属性指定默认直接 TCP 选项的支持 ([#299](https://github.com/Azure/azure-cosmosdb-java/pull/299))
- 改进了直接 TCP 传输客户端中的诊断功能 ([#287](https://github.com/Azure/azure-cosmosdb-java/pull/287))
- 将 Netty 版本升级到了 4.1.42.Final，并将 netty-tcnative.version 升级到了 2.0.26.Final，以解决直接 TCP SSL 问题 ([#274](https://github.com/Azure/azure-cosmosdb-java/pull/274))

## <a name="263---2019-10-23"></a>2.6.3 - 2019-10-23
- 解决了报告错误（重试策略已损坏）的状态代码：
    - `RequestRateTooLargeException.getStatusCode` 现在会正确返回 `TOO_MANY_REQUESTS`。
    - `ServiceUnavailableException.getStatusCode` 现在会正确返回 `SERVICE_UNAVAILABLE`。
    - commons 和 direct-impl 中的 `DocumentClientExceptionTest` 现在会验证是否为所有 `DocumentClientException` 子类型返回了正确的状态代码。

## <a name="262---2019-10-05"></a>2.6.2 - 2019-10-05
- 修复了将 MaxItemCount 设置为 -1 时的查询故障 ([#261](https://github.com/Azure/azure-cosmosdb-java/issues/261))。
- 修复了有关分区拆分的 NPE bug ([#267](https://github.com/Azure/azure-cosmosdb-java/pull/267))。
- 改进了直接模式下的日志记录。 

## <a name="261---2019-09-04"></a>2.6.1 - 2019-09-04

- 针对查询逻辑的多主数据库区域故障转移修补程序 ([#245](https://github.com/Azure/azure-cosmosdb-java/commit/104b4a3b30ffd7c8add01096ce6a9b6e7a3f6318))
- jackson-databind 2.9.9.3 更新 ([#244](https://github.com/Azure/azure-cosmosdb-java/commit/f8e60604d9e2edb0b3bd56ec7909d249de47af50))
- 直接 TCP 修补程序（内存、性能）及指标接口 ([#242](https://github.com/Azure/azure-cosmosdb-java/commit/61562cd16845e2b249a1d91374797faa3469295f))

## <a name="260---2019-08-01"></a>2.6.0 - 2019-08-01

- 使用新查询计划检索程序检索查询信息 ([#111](https://github.com/Azure/azure-cosmosdb-java/commit/6cdbc90386a343484f37c67bec5ec6321729b60a))
- 偏移量限制查询支持 ([#234](https://github.com/Azure/azure-cosmosdb-java/commit/65606c714a6e8019e52889a8b3d49e2a5e0f6a02))

## <a name="251---2019-07-31"></a>2.5.1 - 2019-07-31

- 修复 executeStoredProcedureInternal 以使用客户端重试策略 ([#210](https://github.com/Azure/azure-cosmosdb-java/commit/eb99c15c930a7f3a413445a9e257504468329710))

## <a name="246---2019-07-19"></a>2.4.6 - 2019-07-19

- 内存改进

## <a name="250---2019-06-25"></a>2.5.0 - 2019-06-25

- 默认启用 TCP 传输
- TCP 传输改进
- 大量测试改进

## <a name="245---2019-05-06"></a>2.4.5 - 2019-05-06

- 添加了对 Hash v2 的支持 ([#96](https://github.com/Azure/azure-cosmosdb-java/commit/f780e2d3210fbf875afcdd5e245877a49f7697ee))
- 开源直接连接实现 ([#94](https://github.com/Azure/azure-cosmosdb-java/commit/81a56e9506827e647253b1b6ae39f95a7aee37a3))
- 添加了对诊断字符串的支持 

## <a name="244---2019-06-25"></a>2.4.4 - 2019-06-25

- 其他修补程序

## <a name="243---2019-03-05"></a>2.4.3 - 2019-03-05

- 修复了关闭客户端时资源泄漏的问题

## <a name="242---2019-03-01"></a>2.4.2 - 2019-03-01

- 修复了对跨分区查询的继续标记支持中的 bug

## <a name="241---2019-02-20"></a>2.4.1 - 2019-02-20

- 修复了直接模式下的一些 bug。
- 改进了直接模式下的日志记录。
- 改进了连接管理。

## <a name="240---2019-02-08"></a>2.4.0 - 2019-02-08

- 直接公开上市。
- 添加了对 QueryMetrics 的支持。
- 将接受 java.util.Collection（顺序很重要）的 API 更改为接受 java.util.List。 现在 ConnectionPolicy#getPreferredLocations()、JsonSerialization 和 PartitionKey(.) 接受列表。

## <a name="240-beta1---2019-02-04"></a>2.4.0-beta1 - 2019-02-04

- 添加了对直接 Https 的支持。
- 将接受 java.util.Collection（顺序很重要）的 API 更改为接受 java.util.List。
  现在 ConnectionPolicy#getPreferredLocations()、JsonSerialization 和 PartitionKey(.) 接受列表。
- 修复了网关模式下文档查询的会话 bug。
- 升级了依赖项（netty 0.4.20 [github #79](https://github.com/Azure/azure-cosmosdb-java/issues/79)、RxJava 1.3.8）。

## <a name="231---2019-01-15"></a>2.3.1 - 2019-01-15

- 修复了处理大规模查询响应时的问题。
- 修复了实例化客户端时的资源令牌处理方面的问题 ([github #78](https://github.com/Azure/azure-cosmosdb-java/issues/78))。
- 升级了易受攻击的依赖项 jackson-databind ([github #77](https://github.com/Azure/azure-cosmosdb-java/pull/77))。

## <a name="230---2018-11-29"></a>2.3.0 - 2018-11-29

- 修复了资源泄漏 bug。
- 添加了对 MultiPolygon 的支持
- 在 RequestOptions 中添加了对自定义标头的支持。

## <a name="222---2018-11-26"></a>2.2.2 - 2018-11-26

- 修复了打包 bug。

## <a name="221---2018-11-05"></a>2.2.1 - 2018-11-05

- 修复了写入重试路径中的 NPE bug。
- 修复了终结点管理中的 NPE bug。
- 升级了易受攻击的依赖项 ([github #68](https://github.com/Azure/azure-cosmosdb-java/issues/68))。
- 添加了对 Netty 网络日志记录的支持，以便进行故障排除。

## <a name="220---2018-10-04"></a>2.2.0 - 2018-10-04

- 添加了对多区域写入的支持。

## <a name="210---2018-09-06"></a>2.1.0 - 2018-09-06

- 添加了对代理的支持。
- 添加了对资源令牌授权的支持。
- 修复了处理大分区键时的 bug ([github #63](https://github.com/Azure/azure-cosmosdb-java/issues/63))。
- 改进了文档。
- SDK 重构为更细粒度的模块。

## <a name="201---2018-08-16"></a>2.0.1 - 2018-08-16

- 修复了非英语区域设置的 bug ([github #51](https://github.com/Azure/azure-cosmosdb-java/issues/51))。
- 为冲突资源添加了帮助器方法。

## <a name="200---2018-06-20"></a>2.0.0 - 2018-06-20

- 由于性能原因和授权，由 jackson 替换了 org.json 依赖项 ([github #29](https://github.com/Azure/azure-cosmosdb-java/issues/29))。
- 删除了弃用的 OfferV2 类。
- 向 Offer 类添加了访问器方法以获取吞吐量内容。
- Document/Resource 中返回 org.json 类型的任何方法均已更改为返回 jackson 对象类型。
- 扩展了 JsonSerializable 的类的 getObject(.) 方法已更改为返回 jackson ObjectNode 类型。
- getCollection(.) 方法已更改为返回 ObjectNode 的集合。
- 使用 org.json.JSONObject arg 删除了 JsonSerializable 子类的构造函数。
- JsonSerializable.toJson (SerializationFormattingPolicy.Indented) 现在使用两个空格进行缩进。

## <a name="102---2018-05-18"></a>1.0.2 - 2018-05-18

- 添加了对唯一索引策略的支持。
- 在源选项中添加了对限制响应继续标记大小的支持。
- 在跨分区查询中添加了对分区拆分的支持。
- 修复了 Json 时间戳序列化中的一个 bug ([github #32](https://github.com/Azure/azure-cosmosdb-java/issues/32))。
- 修复了 Json 枚举序列化中的一个 bug。
- 修复了管理 2MB 大小文档中的一个 bug ([github #33](https://github.com/Azure/azure-cosmosdb-java/issues/33))。
- 由于 bug ([jackson-databind: github #1599](https://github.com/FasterXML/jackson-databind/issues/1599))，依赖项 com.fasterxml.jackson.core:jackson-databind 升级到了 2.9.5
- 由于 bug ([rxjava-extras: github #30](https://github.com/davidmoten/rxjava-extras/issues/30))，rxjava-extras 的依赖项升级到了 0.8.0.17。
- pom 文件中的元数据说明更新为与文档的其余部分内联。
- 语法改进 ([github #41](https://github.com/Azure/azure-cosmosdb-java/issues/41))、([github #40](https://github.com/Azure/azure-cosmosdb-java/issues/40))。

## <a name="101---2018-04-20"></a>1.0.1 - 2018-04-20

- 在查询中添加了反压力支持。
- 在查询中添加了对分区键范围 ID 的支持。
- 更改为允许在请求标头中使用更大的继续标记 (bugfix github #24)。
- Netty 依赖项已升级到 4.1.22.Final，以确保 JVM 在主线程完成后关闭。
- 已更改为避免在读取主资源时传递会话令牌。
- 添加了更多示例。
- 添加了更多基准测试方案。
- 修复了 Java 头文件以便生成正确的 javadoc。

## <a name="100---2018-02-26"></a>1.0.0 - 2018-02-26

- 版本 1.0.0 对于在网关模式下使用 Netty 库的非阻止 IO 提供完全的端到端支持。
- 已删除 `azure-documentdb` SDK 的依赖项。
- 项目 ID 在 0.9.0-rc2 中已从 `azure-documentdb-rx` 更改为 `azure-cosmosdb`。
- Java 包名称在 0.9.0-rc2 中已从 `com.microsoft.azure.documentdb` 更改为 `com.microsoft.azure.cosmosdb`。

## <a name="090-rc2---2018-02-26"></a>0.9.0-rc2 - 2018-02-26

- `FeedResponsePage` 已重命名为 `FeedReponse`
- 对 `ConnectionPolicy` 配置的一些小修改。
  ConnectionPolicy 中的所有时间字段和方法均采用“InMillis”作为后缀，以使时间单位更加精确。
- 已删除 `ConnectionPolicy#setProxy()`。
- `FeedOptions#pageSize` 已重命名为 `FeedOptions#maxItemCount`
- 版本 1.0.0 取代了 0.9.x 版本。

## <a name="090-rc1"></a>0.9.0-rc1

- `azure-documentdb-rx` SDK 的第一版。
- CRUD 文档 API 完全不阻止使用 Netty。 已使用阻止 SDK `azure-documentdb` 将查询异步 API 实现为包装器。

## <a name="release-and-retirement-dates"></a>发布日期和停用日期
Azure 会在停用 SDK 时至少提前 12 个月发出通知，以便用户顺利转换为更高版本/受支持版本。

新特性、功能和优化仅添加到最新的 SDK 中。 因此建议你始终尽早升级到最新的 SDK 版本。

使用已停用的 SDK 对 Cosmos DB 发出的任何请求都会被服务拒绝。

> [!WARNING]
> Async Java SDK for SQL API 的所有 1.x 版本将于 2020 年 8 月 30 日停用 。
> 
>
<br/>

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| 2.6.4 |2020 年 1 月 2 日|--- |
| 2.6.3 |2019 年 10 月 23 日|--- |
| 2.6.2 |2019 年 10 月5 日|--- |
| 2.6.1 |2019 年 9 月 4 日|--- |
| 2.6.0 |2019 年 8 月 1 日|--- |
| 2.5.1 |2019 年 7 月 31 日|--- |
| 2.4.6 |2019 年 7 月 19 日|--- |
| 2.5.0 |2019 年 6 月 25 日|--- |
| 2.4.5 |2019 年 5 月 6 日|--- |
| 2.4.3 |2019 年 3 月 5 日|--- |
| 2.4.2 |2019 年 3 月 1 日|--- |
| 2.4.1 |2019 年 2 月 20 日|--- |
| 2.4.0 |2019 年 2 月 8 日|--- |
| 2.4.0-beta-1 |2019 年 2 月 4 日|--- |
| 2.3.1 |2019 年 1 月 15 日|--- |
| 2.3.0 |2018 年 11 月 29 日|--- |
| 2.2.2 |2018 年 11 月 8 日|--- |
| 2.2.1 |2018 年 11 月 2日|--- |
| 2.2.0 |2018 年 9 月 22 日|--- |
| 2.1.0 |2018 年 9 月 5 日|--- |
| 2.0.1 |2018 年 8 月 16 日|--- |
| 2.0.0 |2018 年 6 月 20 日|--- |
| 1.0.2 |2018 年 5 月 18日|2020 年 8 月 30 日 |
| 1.0.1 |2018 年 4 月 20 日|2020 年 8 月 30 日 |
| 1.0.0 |2018 年 2 月 27 日|2020 年 8 月 30 日 |

## <a name="faq"></a>常见问题解答
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另请参阅
若要了解有关 Cosmos DB 的详细信息，请参阅 [Azure Cosmos DB](https://www.azure.cn/home/features/cosmos-db/) 服务页。

<!-- Update_Description: update meta properties, wording update, update link -->