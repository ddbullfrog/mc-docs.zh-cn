---
title: Azure Cosmos DB：SQL .NET API、SDK 和资源
description: 了解有关 SQL .NET API 和 SDK 的所有信息，包括发布日期、停用日期和 Azure Cosmos DB .NET SDK 各版本之间所做的更改。
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
origin.date: 08/12/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.custom: devx-track-dotnet
ms.openlocfilehash: face66316c186220bad1af7e6c813b9bcddf7851
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246357"
---
# <a name="azure-cosmos-db-net-sdk-v2-for-sql-api-download-and-release-notes"></a>适用于 SQL API 的 Azure Cosmos DB .NET SDK v2：下载和发行说明
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

| |  |
|---|---|
|**SDK 下载**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)|
|**API 文档**|[ 参考文档](https://docs.azure.cn/dotnet/api/overview/cosmosdb)|
|**示例**|[.NET代码示例](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples)|
|**入门**|[Azure Cosmos DB .NET SDK 入门](sql-api-get-started.md)|
|**Web 应用教程**|[使用 Azure Cosmos DB 开发 Web 应用程序](sql-api-dotnet-application.md)|
|**当前受支持的框架**|[Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)|

## <a name="release-notes"></a>发行说明

> [!NOTE]
> 如果使用的是 .NET Framework，请参阅面向 .NET Standard 的 [.NET SDK](sql-api-sdk-dotnet-standard.md) 最新版本 3.x。

以下 .NET SDK 子版本可在 2.x.x 版本下使用：

<a name="2.11.6"></a>
### <a name="2116"></a>2.11.6

* 修复了问题：对存储过程、触发器和用户定义的函数执行 CRUD 操作时发生“请求标头太长”的错误

<a name="2.11.5"></a>
### <a name="2115"></a>2.11.5

* 改进了对查询操作期间区域故障转移情况的检测。
* 提高了暂时性 ReadSessionNotAvailable 情形时的复原能力。 

<a name="2.11.4"></a>
### <a name="2114"></a>2.11.4

* 修复连接策略中的 PortReuseMode 设置，使 DocumentClient 遵循该设置。

<a name="2.11.3"></a>
### <a name="2113"></a>2.11.3

* 为连接到特定区域时发生暂时性连接问题的请求添加了跨区域重试机制。 此重试机制使用（且需要）用户定义的 `ConnectionPolicy.PreferredLocations` 首选项列表。
* 已将诊断信息添加到操作取消的异常。

<a name="2.11.2"></a>
### <a name="2112"></a>2.11.2

* 修复查询 FeedOption 中的用户会话令牌，以实现请求级别的会话一致性

<a name="2.11.1"></a>
### <a name="2111"></a>2.11.1

* 修复与 ConnectionPolicy.EnableTcpConnectionEndpointRediscovery 的使用相关的 CPU 问题
* 修复 PartitionKey 在 GROUP BY 查询上未正确传递的问题
* 通过添加 Task.Yeild 来修复重试方案中潜在的 StackOverflowException。 [.NET Core 3 的相关修复](https://github.com/dotnet/coreclr/pull/23152)

<a name="2.11.0"></a>
### <a name="2110"></a>2.11.0

* 将 RequestDiagnosticsString 属性添加到 StoredProcedureResponse
* 进行修复以提高应用于 ConnectionPolicy.SetCurrentLocation 的启发式方法的准确性
* 在 .NET Core 上使用“直接 + TCP”模式时出现的连接错误现在包括 CPU 使用历史记录。

<a name="2.10.3"></a>
### <a name="2103"></a>2.10.3

* 修复了在已关闭的连接的边缘情况下 TCP 连接关闭引发的套接字异常

<a name="2.10.1"></a>
### <a name="2101"></a>2.10.1

* 修复了执行查询时的空引用异常。 此修补程序修复了当未找到 Microsoft.Azure.Documents.ServiceInterop.dll 时回退逻辑中的 bug。

<a name="2.10.0"></a>
### <a name="2100"></a>2.10.0

* 添加了对创建几何图形集合的支持
* 添加了为几何空间索引指定边界框的支持
* 已将 EnableTcpConnectionEndpointRediscovery 添加到 ConnectionPolicy 中，该策略可根据连接关闭通知使地址无效
* 修复了聚合查询 RuntimeBinderException“无法转换类型”
* 修复了权限序列化以包括令牌
* 改进了针对“SDK 无法连接到特定终结点并收到 HttpException”的暂时性区域故障的重试逻辑。
* 通过将默认 RequestTimeout 从 60 秒减少到 10 秒来降低延迟

<a name="2.9.4"></a>
### <a name="294"></a>2.9.4

* 修复了分区键不适用于非 Windows x64 客户端的问题

<a name="2.9.3"></a>
### <a name="293"></a>2.9.3

* 修复了直接 TCP 模式下的计时器池泄漏
* 修复了文档中断开的链接
* 过大的标头现在可跟踪标头大小
* 通过在获取查询计划调用中排除会话令牌减小了标头大小

<a name="2.9.2"></a>
### <a name="292"></a>2.9.2

* 修复了 order by 继续标记中的非 ASCII 字符

<a name="2.9.1"></a>
### <a name="291"></a>2.9.1

* 修复了 Microsoft.Azure.Documents.ServiceInterop.dll 正常回退 bug [问题 #750](https://github.com/Azure/azure-cosmos-dotnet-v2/issues/750)

<a name="2.9.0"></a>
### <a name="290"></a>2.9.0

* 添加了对 [GROUP BY](/cosmos-db/sql-query-group-by) 查询的支持
* 查询现在在执行之前会检索查询计划，以确保单分区查询和跨分区查询之间的行为一致。

<a name="2.8.1"></a>
### <a name="281"></a>2.8.1

* 已将 RequestDiagnosticsString 添加到 FeedResponse
* 修复了更新插入和替换文档的序列化设置

<a name="2.7.0"></a>
### <a name="270"></a>2.7.0

* 在 order by 查询中添加了对数组和对象的支持
* 处理了有效的分区键冲突
* 使用 ThenBy 运算符添加了对多个 OrderBy 运算符的 LINQ 支持
* 修复了 AysncCache 死锁问题，以便它可以使用单线程任务计划程序

<a name="2.6.0"></a>
### <a name="260"></a>2.6.0

* 已将 PortReusePolicy 添加到 ConnectionPolicy
* 修复了在 UWP 应用中使用 SDK 时出现的 ntdll!RtlGetVersion TypeLoadException 问题

<a name="2.5.1"></a>
### <a name="251"></a>2.5.1

* SDK 的 System.Net.Http 版本现在与 NuGet 包中定义的版本匹配
* 允许写入请求回退到不同的区域（如果原始区域发生故障）
* 为写入请求添加了会话重试策略

<a name="2.4.4"></a>
### <a name="242"></a>2.4.2

* 已使程序集版本和文件版本与 nuget 包版本相同。

<a name="2.4.1"></a>
### <a name="241"></a>2.4.1

* 针对导致空页的查询修复了跟踪争用条件

<a name="2.4.0"></a>
### <a name="240"></a>2.4.0

* 增加了 LINQ 查询的十进制精度大小。
* 添加了 CompositePath、CompositePathSortOrder、SpatialSpec、SpatialType 和 PartitionKeyDefinitionVersion 这几种新类
* 已将 TimeToLivePropertyPath 添加到 DocumentCollection
* 已将 CompositeIndexes 和 SpatialIndexes 添加到 IndexPolicy
* 已将 Version 添加到 PartitionKeyDefinition
* 已将 None 添加到 PartitionKey
* 修复了 bug 以正确处理会导致 JsonReaderException 的非 JSON 有效负载

<a name="2.3.0"></a>
### <a name="230"></a>2.3.0

* 已将 IdleTcpConnectionTimeout、OpenTcpConnectionTimeout、MaxRequestsPerTcpConnection 和 MaxTcpConnectionsPerEndpoint 添加到 ConnectionPolicy。

<a name="2.2.3"></a>
### <a name="223"></a>2.2.3

* 诊断改进

<a name="2.2.2"></a>
### <a name="222"></a>2.2.2

* 添加了环境变量设置“POCOSerializationOnly”。
* 删除了 DocumentDB.Spatial.Sql.dll，现在在 Microsoft.Azure.Documents.ServiceInterop.dll 中包含

<a name="2.2.1"></a>
### <a name="221"></a>2.2.1

* 对 StoredProcedure 执行调用故障转移期间的重试逻辑进行了改进。

* 将 DocumentClientEventSource 设为单一实例。 

* 修复了 GatewayAddressCache 超时不遵守 ConnectionPolicy RequestTimeout 的问题。

<a name="2.2.0"></a>
### <a name="220"></a>2.2.0

* 对于直接/TCP 传输诊断，添加了 TransportException，这是该 SDK 的一个内部异常类型。 出现在异常消息中时，此类型会输出附加信息，以便排查客户端连接问题。

* 添加了新的构造函数重载，它采用 HttpMessageHandler，后者是用于发送 HttpClient 请求的 HTTP 处理程序堆栈（例如，HttpClientHandler）。

* 修复了无法正确处理带有空值的标头的 bug。

* 改进了集合缓存验证。

<a name="2.1.3"></a>
### <a name="213"></a>2.1.3

* 更新到 4.3.2 版 System.Net.Security。

<a name="2.1.2"></a>
### <a name="212"></a>2.1.2

* 诊断跟踪改进。

<a name="2.1.1"></a>
### <a name="211"></a>2.1.1

* 为多区域请求暂时性故障添加了更多复原能力。   

<a name="2.1.0"></a>
### <a name="210"></a>2.1.0

* 添加了多区域写入支持。
* 使用 TOP 改进了跨分区查询性能。
* 修复了不遵守 MaxBufferedItemCount 导致内存不足问题的 bug。

<a name="2.0.0"></a>
### <a name="200"></a>2.0.0

* 添加了请求取消支持。
* 将 SetCurrentLocation 添加到 ConnectionPolicy，它会根据区域自动填充首选位置。
* 修复了具有 Min/Max 以及在单个分区上没有文档匹配的筛选的跨分区查询中的 Bug。
* DocumentClient 方法现在与 IDocumentClient 具有奇偶校验。
* 更新了直接 TCP 传输堆栈以减少建立的连接数。
* 为非 Windows 客户端添加了对 Direct Mode TCP 的支持。

<a name="2.0.0-preview2"></a>
### <a name="200-preview2"></a>2.0.0-preview2

* 添加了请求取消支持。
* 将 SetCurrentLocation 添加到 ConnectionPolicy，它会根据区域自动填充首选位置。
* 修复了具有 Min/Max 以及在单个分区上没有文档匹配的筛选的跨分区查询中的 Bug。

<a name="2.0.0-preview"></a>
### <a name="200-preview"></a>2.0.0-preview

* DocumentClient 方法现在与 IDocumentClient 具有奇偶校验。
* 更新了直接 TCP 传输堆栈以减少建立的连接数。
* 为非 Windows 客户端添加了对 Direct Mode TCP 的支持。

<a name="1.22.0"></a>
### <a name="1220"></a>1.22.0

* 将 ConsistencyLevel 属性添加到了 FeedOptions。
* 将 JsonSerializerSettings 添加到了 RequestOptions 和 FeedOptions。
* 将 EnableReadRequestsFallback 添加到了 ConnectionPolicy。

<a name="1.21.1"></a>
### <a name="1211"></a>1.21.1

* 修复了临界情况下跨分区 order by 查询出现的 KeyNotFoundException。
* 修复了在 LINQ 查询的 select 子句中不接受 JsonPropery 属性的 bug。

<a name="1.20.2"></a>
### <a name="1202"></a>1.20.2

* 修复了在某些争用情况下出现的 bug，该 bug 在使用会话一致性级别时导致间歇性错误“Microsoft.Azure.Documents.NotFoundException:读取会话不可用于输入会话令牌”。

<a name="1.20.1"></a>
### <a name="1201"></a>1.20.1

* 改进了在 FeedOptions 中将 MaxDegreeOfParallelism 属性设置为“-1”时的跨分区查询性能。
* 向 QueryMetrics 添加了新的 ToString() 函数。
* 公开了有关读取集合的分区统计信息。
* 向 ChangeFeedOptions 添加了 PartitionKey 属性。
* 小 bug 修复。

<a name="1.19.1"></a>
### <a name="1191"></a>1.19.1

* 添加了通过使用 DocumentCollection 上的 UniqueKeyPolicy 属性来指定文档唯一索引的功能。
* 修复了自定义 JsonSerializer 设置无法用于某些查询和存储过程执行的 bug。

<a name="1.19.0"></a>
### <a name="1190"></a>1.19.0

* 在 API 参考文档、程序集中的元数据信息和 NuGet 包中品牌从 Azure DocumentDB 更改为 Azure Cosmos DB。 
* 通过以直接连接模式发送的请求的响应公开诊断信息和延迟。 在 ResourceResponse 类中属性名称是 RequestDiagnosticsString 和 RequestLatency。
* 此 SDK 版本需要最新版本的 Azure Cosmos DB 模拟器（可从 https://aka.ms/cosmosdb-emulator 下载）。

<a name="1.18.1"></a>
### <a name="1181"></a>1.18.1

* 对 Azure 友元程序集进行了内部更改。

<a name="1.18.0"></a>
### <a name="1180"></a>1.18.0

* 添加了多项可靠性修复和改进。

<a name="1.17.0"></a>
### <a name="1170"></a>1.17.0

* 添加了 PartitionKeyRangeId 的支持作为 FeedOption，将查询结果限定在某个特定分区键范围值。 
* 添加了 StartTime 的支持作为 ChangeFeedOption，以开始查找该时间后的更改。 

<a name="1.16.1"></a>
### <a name="1161"></a>1.16.1
- 修复了 JsonSerializable 类中可能引起堆栈溢出异常的问题。 

<a name="1.16.0"></a>
### <a name="1160"></a>1.16.0
- 修复了在因在 DocumentClient 构造函数中引入 JsonSerializerSettings 作为可选参数而需要重新编译的问题。
- 将 DocumentClient 构造函数标记为已过时，需要 JsonSerializerSettings 作为最后一个参数，以在传入 JsonSerializerSettings 参数时允许 ConnectionPolicy 和 ConsistencyLevel 参数的默认值。

<a name="1.15.0"></a>
### <a name="1150"></a>1.15.0
- 添加了对在实例化 DocumentClient 时指定自定义 JsonSerializerSettings 的支持。

<a name="1.14.1"></a>
### <a name="1141"></a>1.14.1
- 修复了影响 x64 计算机的一个问题，该计算机不支持 SSE4 指令，并在运行 Azure Cosmos DB DocumentDB API 查询时引发 SEHException。

<a name="1.14.0"></a>
### <a name="1140"></a>1.14.0
- 添加了对每分钟请求单位数 (RU/m) 功能的支持。
- 添加了对称为“ConsistentPrefix”的新一致性级别的支持。
- 添加了对查询单个分区的指标的支持。
- 添加了对限制查询的继续标记大小的支持。
- 添加了对失败请求的更详细跟踪的支持。
- 在 SDK 中进行了一些性能改进。

<a name="1.13.4"></a>
### <a name="1134"></a>1.13.4
- 与 1.13.3 功能相同。 进行了一些内部更改。

<a name="1.13.3"></a>
### <a name="1133"></a>1.13.3
- 在功能上与 1.13.2 相同。 进行了一些内部更改。

<a name="1.13.2"></a>
### <a name="1132"></a>1.13.2
- 修复了忽略 FeedOptions 中为聚合查询提供的 PartitionKey 值的问题。
- 修复了在中途跨分区执行 OrderBy 查询期间透明处理分区管理的问题。

<a name="1.13.1"></a>
### <a name="1131"></a>1.13.1
- 修复了在 ASP.NET 上下文内使用时，在某些异步 API 中导致死锁的问题。

<a name="1.13.0"></a>
### <a name="1130"></a>1.13.0
- 修复程序，用于使 SDK 更具弹性，以便在某些情况下自动故障转移。

<a name="1.15.0"></a>
### <a name="1150"></a>1.15.0
- 修复了偶尔会导致“WebException:无法解析远程名称”的问题。
- 通过向 ReadDocumentAsync API 添加新重载，添加了对直接读取类型化文档的支持。

<a name="1.12.1"></a>
### <a name="1121"></a>1.12.1
- 添加了对聚合查询（COUNT、MIN、MAX、SUM 和 AVG）的 LINQ 支持。
- 修复了由于使用了事件处理程序而导致的 ConnectionPolicy 对象的内存泄漏问题。
- 修复了使用 ETag 时 UpsertAttachmentAsync 不正常工作的问题。
- 修复了对字符串字段进行排序时跨分区按查询条件排序不正常工作的问题。

<a name="1.12.0"></a>
### <a name="1120"></a>1.12.0
- 添加了对聚合查询（COUNT、MIN、MAX、SUM、AVG）的支持。 请参阅[聚合支持](/cosmos-db/sql-query-aggregates)。
- 将分区集合上的最小吞吐量从 10,100 RU/s 降低到 2500 RU/s。

<a name="1.11.4"></a>
### <a name="1114"></a>1.11.4

- 针对 32 位主机进程中某些分区查询的失败问题进行了修复。
- 修复了以下问题：网关模式下，会话容器中不更新失败请求的标记。
- 修复了以下问题：投影中调用 UDF 的查询在某些情况下失败。

<a name="1.11.3"></a>
### <a name="1113"></a>1.11.3

- 针对使用令牌更新会话容器的请求失败问题进行了修复。 
- 添加了支持，使该 SDK 可用于 32 位主机进程。 请注意，如果使用跨分区查询，建议使用 64 位主机进程以提高性能。
- 改进了性能，可在 IN 表达式中使用大量分区键值进行查询。

<a name="1.11.1"></a>
### <a name="1111"></a>1.11.1

- 对 1.11.0 版本中引入的 CreateDocumentCollectionIfNotExistsAsync API 的轻微性能修复。 
- 性能修复，针对 SDK 中涉及高度并发请求的方案。

<a name="1.11.0"></a>
### <a name="1110"></a>1.11.0

- 支持新类和方法，可处理集合内的文档[更改源](/documentdb/documentdb-change-feed)。 
- 支持跨分区查询延续和跨分区查询的一些性能改进。
- 添加 CreateDatabaseIfNotExistsAsync 和 CreateDocumentCollectionIfNotExistsAsync 方法。
- 针对以下系统函数提供了 LINQ 支持：IsDefined、IsNull 和 IsPrimitive。
- 修复的问题：在 Nuget 包与具有 project.json 工具的项目配合使用时，Microsoft.Azure.Documents.ServiceInterop.dll 和 DocumentDB.Spatial.Sql.dll 程序集自动 binplace 到应用程序的 bin 文件夹中。
- 支持发出客户端侧 ETW 跟踪，这对调试方案很有用。

<a name="1.10.0"></a>
### <a name="1100"></a>1.10.0

- 添加已分区集合得直接连接支持。
- 改进了受限停滞一致性级别的性能。
- 添加了转换谓词时对 StringEnumConverter、IsoDateTimeConverter 和 UnixDateTimeConverter 的 LINQ 支持。
- 各种 SDK bug 修复。

<a name="1.9.5"></a>
### <a name="195"></a>1.9.5

- 修复了导致以下 NotFoundException 的问题：读取会话不可用于输入会话令牌。 在地理分散的帐户的读取区域查询时，在某些情况下会发生此异常。
- 已公开 ResourceResponse 类中的 ResponseStream 属性，使用该属性可直接访问响应中的基础流。 

<a name="1.9.4"></a>
### <a name="194"></a>1.9.4

- 修改了 ResourceResponse、FeedResponse、StoredProcedureResponse 和 MediaResponse 类，以实现相应的公共接口，以便可以模拟它们进行测试驱动的部署 (TDD)。
- 解决了使用自定义 JsonSerializerSettings 对象序列化数据时导致格式错误的分区键标头的问题。

<a name="1.9.3"></a>
### <a name="193"></a>1.9.3

- 修复了一个问题，该问题导致长时间运行的查询失败并显示以下错误：授权令牌当前无效。
- 解决了从跨分区 top/order-by 查询中删除了原始 SqlParameterCollection 的问题。

<a name="1.9.2"></a>
### <a name="192"></a>1.9.2

- 已对分区集合添加并行查询支持。
- 已对分区集合添加跨分区 ORDER BY 和 TOP 查询支持。
- 已修复当使用对 DocumentDB Nuget 包的引用来引用 DocumentDB 项目时缺少所需的 DocumentDB.Spatial.Sql.dll 和 Microsoft.Azure.Documents.ServiceInterop.dll 引用的错误。
- 修复了在 LINQ 中使用用户定义的函数时无法使用不同类型的参数的错误。 
- 修复了多区域复制帐户的 bug，此 bug 会使 Upsert 调用定向至读取位置而非写入位置。
- 已将缺少的方法添加到了 IDocumentClient 接口： 
    - 采用 mediaStream 和选项作为参数的 UpsertAttachmentAsync 方法。
    - 采用选项作为参数的 CreateAttachmentAsync 方法。
    - 采用 querySpec 作为参数的 CreateOfferQuery 方法。

- 已解封 IDocumentClient 接口中公开的公共类。

<a name="1.8.0"></a>
### <a name="180"></a>1.8.0

- 添加了对多区域数据库帐户的支持。
- 添加了对限制请求重试的支持。用户可以通过配置 ConnectionPolicy.RetryOptions 属性来自定义重试次数和最长等待时间。
- 添加新的 IDocumentClient 接口，用于定义所有 DocumenClient 属性和方法的签名。 在做出此项更改的同时，已将用于创建 IQueryable 和 IOrderedQueryable 的扩展方法更改为 DocumentClient 类本身的方法。
- 添加了配置选项用于设置给定 DocumentDB 终结点 URI 的 ServicePoint.ConnectionLimit。 使用 ConnectionPolicy.MaxConnectionLimit 可以更改默认值（设置为 50）。
- 已弃用 IPartitionResolver 及其实现。 对 IPartitionResolver 的支持现已过时。 建议使用分区集合来提高存储和吞吐量。

<a name="1.7.1"></a>
### <a name="171"></a>1.7.1

- 向采用 RequestOptions 作为参数的基于 Uri 的 ExecuteStoredProcedureAsync 方法添加了重载。

<a name="1.7.0"></a>
### <a name="170"></a>1.7.0

- 对文档添加了生存时间 (TTL) 支持。

<a name="1.6.3"></a>
### <a name="163"></a>1.6.3

- 修复了 .NET SDK 的 Nuget 打包（用于将其打包为 Azure 云服务解决方案的一部分）中的 bug。

<a name="1.6.2"></a>
### <a name="162"></a>1.6.2

- 实现了[分区集合](/cosmos-db/partitioning-overview)和用户定义的性能级别。 

<a name="1.5.3"></a>
### <a name="153"></a>1.5.3

- [已修复] 查询 DocumentDB 终结点时引发：“System.Net.Http.HttpRequestException:将内容复制到流时出错。

<a name="1.5.2"></a>
### <a name="152"></a>1.5.2

- 扩展的 LINQ 支持，包括用于分页、条件表达式和范围比较的新运算符。 
    - Take 运算符，用于在 LINQ 中启用 SELECT TOP 行为。
    - CompareTo 运算符，用于启用字符串范围比较。
    - 条件运算符 (?) 和 coalesce 运算符 (??)。

- **[已修复]** 合并模型投影与 linq 查询中的 Where-In 时出现 ArgumentOutOfRangeException。 [#81](https://github.com/Azure/azure-documentdb-dotnet/issues/81)

<a name="1.5.1"></a>
### <a name="151"></a>1.5.1

- **[已修复]** 如果 Select 不是最后一个表达式，则 LINQ 提供程序假定没有投影，并且错误地生成 SELECT *。 [#58](https://github.com/Azure/azure-documentdb-dotnet/issues/58)

<a name="1.5.0"></a>
### <a name="150"></a>1.5.0

- 实现了 Upsert，添加了 UpsertXXXAsync 方法。
- 针对所有请求进行了性能改进。
- LINQ 提供程序支持字符串的条件表达式、联合表达式和 CompareTo 方法。
- [已修复] LINQ 提供程序 --> 在列表上实现 Contains 方法以生成与 IEnumerable 和 Array 上相同的 SQL。
- [已修复] BackoffRetryUtility 再次使用相同的 HttpRequestMessage 而不是在重试时创建一个新的。
- [已过时] UriFactory.CreateCollection --> 现在应使用 UriFactory.CreateDocumentCollection。

<a name="1.4.1"></a>
### <a name="141"></a>1.4.1

- **[已修复]** 使用非 en 区域性信息（如 nl-NL 等）时出现本地化问题。 

<a name="1.4.0"></a>
### <a name="140"></a>1.4.0
- 基于 ID 的路由 
    - 新增了用于协助构造基于 ID 的资源链接的 UriFactory 帮助程序。
    - 在 DocumentClient 上新增了重载以接受 URI。

- 在针对地理空间的 LINQ 中添加了 IsValid() 和 IsValidDetailed()。
- 增强了 LINQ 提供程序支持。
    - **数学** - Abs、Acos、Asin、Atan、Ceiling、Cos、Exp、Floor、Log、Log10、Pow、Round、Sign、Sin、Sqrt、Tan、Truncate。
    - **字符串** - Concat、Contains、EndsWith、IndexOf、Count、ToLower、TrimStart、Replace、Reverse、TrimEnd、StartsWith、SubString、ToUpper。
    - **数组** - Concat、Contains、Count。
    - **IN** 运算符。

<a name="1.3.0"></a>
### <a name="130"></a>1.3.0
- 现已开始支持修改索引策略。
    - 在 DocumentClient 中新增了 ReplaceDocumentCollectionAsync 方法。
    - 在 ResourceResponse 中新增了 IndexTransformationProgress 属性，用于跟踪索引策略更改的百分比进度。
    - DocumentCollection.IndexingPolicy 现在是可变的。

- 添加了对空间索引和查询的支持 
    - 新增了 Microsoft.Azure.Documents.Spatial 命名空间，用于序列化/反序列化 Point 和 Polygon 等空间类型。
    - 新增了 SpatialIndex 类，用于为存储在 DocumentDB 中的 GeoJSON 数据编制索引。

- **[已修复]** ：从 linq 表达式生成的不正确的 SQL 查询。 [#38](https://github.com/Azure/azure-documentdb-dotnet/issues/38)

<a name="1.2.0"></a>
### <a name="120"></a>1.2.0
- 对 Newtonsoft.Json v5.0.7 的依赖关系。

- 更改为支持 Order By：
    - LINQ 提供程序支持 OrderBy() 或 OrderByDescending()。
    - 支持 Order By 的 IndexingPolicy。

    **NB：可能推出的中断性变更** 

    如果现有代码使用自定义索引策略预配集合，那么现有代码将需要更新为支持新的 IndexingPolicy 类。 如果没有任何自定义索引策略，此更改不会影响你。

<a name="1.1.0"></a>
### <a name="110"></a>1.1.0
- 通过使用新的 HashPartitionResolver 和 RangePartitionResolver 类以及 IPartitionResolver 来支持对数据进行分区。
- DataContract 序列化。
- LINQ 提供程序中的 Guid 支持。
- LINQ 中的 UDF 支持。

<a name="1.0.0"></a>
### <a name="100"></a>1.0.0
- GA SDK。

## <a name="release--retirement-dates"></a>发布和停用日期

Azure 会在停用 SDK 时至少提前 12 个月发出通知，以便用户顺利转换为更高版本/受支持版本。 新特性和功能以及优化仅添加到当前 SDK，因此建议始终尽早升级到最新的 SDK 版本。 

在 2022 年 8 月 31 日之后，Azure Cosmos DB 将不再进行 bug 修复，不再添加新功能，也不再支持 1.x 版的 Azure Cosmos DB .NET 或 .NET Core SDK for SQL API。 如果你不想升级，则从 1.x 版 SDK 发送的请求将继续由 Azure Cosmos DB 服务处理。

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [2.11.4](#2.11.4) |2020 年 7 月 30 日 |--- |
| [2.11.3](#2.11.3) |2020 年 7 月 29 日 |--- |
| [2.11.2](#2.11.2) |2020 年 7 月 14 日 |--- |
| [2.11.1](#2.11.1) |2020 年 7 月 1 日 |--- |
| [2.9.2](#2.9.2) |2019 年 11 月 15 日 |--- |
| [2.9.1](#2.9.1) |2019 年 11 月 13 日 |--- |
| [2.9.0](#2.9.0) |2019 年 10 月 30 日 |--- |
| [2.8.1](#2.8.1) |2019 年 11 月 8 日 |--- |
| [2.7.0](#2.7.0) |2019 年 9 月 23 日 |--- |
| [2.6.0](#2.6.0) |2019 年 8 月 30 日 |--- |
| [2.5.1](#2.5.1) |2019 年 7 月 2 日 |--- |
| [2.4.1](#2.4.1) |2019 年 6 月 20 日 |--- |
| [2.4.0](#2.4.0) |2019 年 5 月 5 日 |--- |
| [2.3.0](#2.3.0) |2019 年 4 月 4 日 |--- |
| [2.2.3](#2.2.3) |2019 年 2 月 11 日 |--- |
| [2.2.2](#2.2.2) |2019 年 2 月 6 日 |--- |
| [2.2.1](#2.2.1) |2018 年 12 月 24 日 |--- |
| [2.2.0](#2.2.0) |2018 年 12 月 7 日 |--- |
| [2.1.3](#2.1.3) |2018 年 10 月 15 日 |--- |
| [2.1.2](#2.1.2) |2018 年 10 月 4 日 |--- |
| [2.1.1](#2.1.1) |2018 年 9 月 27 日 |--- |
| [2.1.0](#2.1.0) |2018 年 9 月 21 日 |--- |
| [2.0.0](#2.0.0) |2018 年 9 月 7 日 |--- |
| [1.22.0](#1.22.0) |2018 年 4 月 19 日 | 2022 年 8 月 31 日 |
| [1.21.1](#1.20.1) |2018 年 3 月 9 日 | 2022 年 8 月 31 日  |
| [1.20.2](#1.20.1) |2018 年 2 月 21 日 | 2022 年 8 月 31 日  |
| [1.20.1](#1.20.1) |2018 年 2 月 5 日 | 2022 年 8 月 31 日  |
| [1.19.1](#1.19.1) |2017 年 11 月 16 日 | 2022 年 8 月 31 日  |
| [1.19.0](#1.19.0) |2017 年 11 月 10 日 | 2022 年 8 月 31 日  |
| [1.18.1](#1.18.1) |2017 年 11 月 7 日 | 2022 年 8 月 31 日  |
| [1.18.0](#1.18.0) |2017 年 10 月 17 日 | 2022 年 8 月 31 日  |
| [1.17.0](#1.17.0) |2017 年 8 月 10 日 | 2022 年 8 月 31 日  |
| [1.16.1](#1.16.1) |2017 年 8 月 7 日 | 2022 年 8 月 31 日  |
| [1.16.0](#1.16.0) |2017 年 8 月 2 日 | 2022 年 8 月 31 日  |
| [1.15.0](#1.15.0) |2017 年 6 月 30 日 | 2022 年 8 月 31 日 |
| [1.14.1](#1.14.1) |2017 年 5 月 23 日 | 2022 年 8 月 31 日  |
| [1.14.0](#1.14.0) |2017 年 5 月 10 日 | 2022 年 8 月 31 日  |
| [1.13.4](#1.13.4) |2017 年 5 月 9 日 | 2022 年 8 月 31 日  |
| [1.13.3](#1.13.3) |2017 年 5 月 6 日 | 2022 年 8 月 31 日  |
| [1.13.2](#1.13.2) |2017 年 4 月 19 日 | 2022 年 8 月 31 日  |
| [1.13.1](#1.13.1) |2017 年 3 月 29 日 | 2022 年 8 月 31 日  |
| [1.13.0](#1.13.0) |2017 年 3 月 24 日 | 2022 年 8 月 31 日  |
| [1.12.1](#1.12.1) |2017 年 3 月 14 日 | 2022 年 8 月 31 日  |
| [1.12.0](#1.12.0) |2017 年 2 月 15 日 | 2022 年 8 月 31 日  |
| [1.11.4](#1.11.4) |2017 年 2 月 6 日 | 2022 年 8 月 31 日  |
| [1.11.3](#1.11.3) |2017 年 1 月 26 日 | 2022 年 8 月 31 日  |
| [1.11.1](#1.11.1) |2016 年 12 月 21 日 | 2022 年 8 月 31 日  |
| [1.11.0](#1.11.0) |2016 年 12 月 08 日 | 2022 年 8 月 31 日  |
| [1.10.0](#1.10.0) |2016 年 9 月 27 日 | 2022 年 8 月 31 日  |
| [1.9.5](#1.9.5) |2016 年 9 月 1 日 | 2022 年 8 月 31 日  |
| [1.9.4](#1.9.4) |2016 年 8 月 24 日 | 2022 年 8 月 31 日  |
| [1.9.3](#1.9.3) |2016 年 8 月 15 日 | 2022 年 8 月 31 日 |
| [1.9.2](#1.9.2) |2016 年 7 月 23 日 | 2022 年 8 月 31 日  |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 | 2022 年 8 月 31 日  |
| [1.7.1](#1.7.1) |2016 年 5 月 6 日 | 2022 年 8 月 31 日  |
| [1.7.0](#1.7.0) |2016 年 4 月 26 日 | 2022 年 8 月 31 日  |
| [1.6.3](#1.6.3) |2016 年 4 月 8 日 | 2022 年 8 月 31 日  |
| [1.6.2](#1.6.2) |2016 年 3 月 29 日 | 2022 年 8 月 31 日  |
| [1.5.3](#1.5.3) |2016 年 2 月 19 日 | 2022 年 8 月 31 日 |
| [1.5.2](#1.5.2) |2015 年 12 月 14 日 | 2022 年 8 月 31 日  |
| [1.5.1](#1.5.1) |2015 年 11 月 23 日 | 2022 年 8 月 31 日  |
| [1.5.0](#1.5.0) |2015 年 10 月 5 日 | 2022 年 8 月 31 日  |
| [1.4.1](#1.4.1) |2015 年 8 月 25 日 | 2022 年 8 月 31 日  |
| [1.4.0](#1.4.0) |2015 年 8 月 13 日 | 2022 年 8 月 31 日  |
| [1.3.0](#1.3.0) |2015 年 8 月 5 日 | 2022 年 8 月 31 日 |
| [1.2.0](#1.2.0) |2015 年 7 月 6 日 | 2022 年 8 月 31 日  |
| [1.1.0](#1.1.0) |2015 年 4 月 30 日 | 2022 年 8 月 31 日  |
| [1.0.0](#1.0.0) |2015 年 4 月 8 日 |  2022 年 8 月 31 日  |

## <a name="faq"></a>常见问题解答

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另请参阅

若要了解有关 Cosmos DB 的详细信息，请参阅 [Azure Cosmos DB](https://www.azure.cn/home/features/cosmos-db/) 服务页。

<!-- Update_Description: update meta properties, wording update, update link -->