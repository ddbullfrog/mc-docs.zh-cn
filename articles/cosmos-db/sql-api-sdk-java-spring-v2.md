---
title: 用于 SQL API 的Spring Data Azure Cosmos DB v2 的发行说明和资源
description: 了解有关用于 SQL API 的 Spring Data Azure Cosmos DB v2 的所有信息，包括发行日期、停用日期和 Azure Cosmos DB SQL Async Java SDK 各版本之间所做的更改。
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
ms.openlocfilehash: 0648c854d79ab56380b4cdd635ffc44265c1806e
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246953"
---
<!--Verified successfully-->
# <a name="spring-data-azure-cosmos-db-v2-for-core-sql-api-release-notes-and-resources"></a>用于 Core (SQL) API 的 Spring Data Azure Cosmos DB v2：发行说明和资源
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

通过用于 Core (SQL) 的 Spring Data Azure Cosmos DB v2，开发人员可以在 Spring 应用程序中使用 Azure Cosmos DB。 Spring Data Azure Cosmos DB 公开 Spring Data 接口，以便操作数据库和集合、使用文档和发出查询。 同一 Maven 项目中同时支持 Sync 和 Async (Reactive) API。 

[Spring Framework](https://spring.io/projects/spring-framework) 是一种简化 Java 应用程序开发的编程和配置模型。 为引用组织的网站，Spring 使用依赖项注入来简化应用程序的“管道”。 由于生成和测试应用程序变得更加简单，因此许多开发人员喜欢 Spring。 [Spring Boot](https://spring.io/projects/spring-boot) 重视 Web 应用程序和微服务的开发，扩展了这种处理管道的理念。 [Spring Data](https://spring.io/projects/spring-data) 是一种编程模型，用于从 Spring 或 Spring Boot 应用程序的上下文中访问数据存储（如 Azure Cosmos DB）。 

可在 [Azure Spring Cloud](https://www.azure.cn/home/features/spring-cloud/) 应用程序中使用 Spring Data Azure Cosmos DB。

> [!IMPORTANT]  
> 这些发行说明适用于 Spring Data Azure Cosmos DB v2。 可以在[此处](sql-api-sdk-java-spring-v3.md)找到 v3 发行说明。 
>
> Spring Data Azure Cosmos DB 仅支持 SQL API。
>

<!--MOONCAKE CORRECT ON THE LINK-->
<!--These links lead to pages of the global-->

> 以下指南支持其他 Azure Cosmos DB API 上的 Spring Data：
> * [将适用于 Apache Cassandra 的 Spring Data 用于 Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [将 Spring Data MongoDB 用于 Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [将 Spring Data Gremlin 用于 Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>
> 希望快速开始？
> 1. 安装[最低支持的 Java 运行时版本，JDK 8](https://docs.microsoft.com/java/azure/jdk/)，以便可以使用 SDK。
> 2. 使用初学者套件创建 Spring Data Azure Cosmos DB 应用 - [非常简单](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db)！
> 3. 通过 [Spring Data Azure Cosmos DB 开发人员指南](https://docs.microsoft.com/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb)了解基本的 Azure Cosmos DB 请求。
>
> 可通过 [Spring Initializr](https://start.spring.io/) 快速启动 Spring Boot Starter 应用！
>

## <a name="helpful-content"></a>帮助性内容

| Content | 链接 |
|---|---|
| **SDK 下载** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/spring-data-cosmosdb) |
|**API 文档** | [Spring Data Azure Cosmos DB 参考文档]() |
|**参与 SDK** | [GitHub 上的 Spring Data Azure Cosmos DB 存储库](https://github.com/microsoft/spring-data-cosmosdb) | 
|**Spring Boot Starter**| [适用于 Java 的 Azure Cosmos DB Spring Boot Starter 客户端库](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-cosmosdb) |
|**用于 Azure Cosmos DB 的 Spring TODO 应用示例**| [应用服务 Linux 中的端到端 Java 体验（第 2 部分）](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2) |
|**开发人员指南** | [Spring Data Azure Cosmos DB 开发人员指南](https://docs.microsoft.com/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) | 
|**Starter 使用指南** | [如何将 Spring Boot Starter 与 Azure Cosmos DB SQL API 配合使用](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db) <br /> [适用于 Azure Spring Boot Starter Cosmos DB 的 GitHub 存储库](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-cosmosdb) |
|**应用服务示例** | [如何将 Spring 和 Cosmos DB 与 Linux 上的应用服务配合使用](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-app-with-cosmos-db-on-app-service-linux) <br /> [TODO 应用示例](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git) |

<!--MOONCAKE CORRECT ON THE LINK-->

## <a name="release-history"></a>版本历史记录

### <a name="230-2020-05-21"></a>2.3.0 (2020-05-21)
#### <a name="new-features"></a>新增功能
* 将 Spring Boot 版本更新为 2.3.0 
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="225-2020-05-19"></a>2.2.5 (2020-05-19)
#### <a name="new-features"></a>新增功能
* 已将 Azure Cosmos DB 版本更新为 v3.7.3
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 包含来自 Cosmos SDK v3.7.3 的内存泄漏修复和 netty 版本升级 

### <a name="224-2020-04-06"></a>2.2.4 (2020-04-06)
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 修复了 CosmosDbConfig 中要考虑的 allowTelemetry 标志
* 修复了容器上的 TTL 属性

### <a name="223-2020-02-25"></a>2.2.3 (2020-02-25)
#### <a name="new-features"></a>新增功能
* 添加了按分区键 API 的新 findAll
* 已将 azure-cosmos 版本更新为 3.7.0
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 修复了 collectionName -> containerName
* 修复了 entityClass & domainClass -> domainType
* 修复了以下问题：返回存储库保存的实体集合而不是输入实体

### <a name="2110-2020-02-25"></a>2.1.10 (2020-02-25)
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 向后移植修复了以下问题：返回存储库保存的实体集合而不是输入实体

### <a name="222-2020-01-15"></a>2.2.2 (2020-01-15)
#### <a name="new-features"></a>新增功能
* 已将 azure-cosmos 版本更新为 v3.6.0
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="221-2019-12-31"></a>2.2.1 (2019-12-31)
#### <a name="new-features"></a>新增功能
* 已将 Cosmos DB SDK 版本更新为 3.5.0
* 添加了注释字段以启用/禁用自动创建集合
* 公开了通过 CosmosDBAccessException 的 CosmosClientException 以便更好地处理异常
* 公开了通过 ResponseDiagnostics 的 requestCharge 和 activityId
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* SDK 3.5.0 更新修复了以下问题：Cosmos DB HTTP 响应标头大于 8192 字节时出现异常；ConsistencyPolicy.defaultConsistencyLevel() 在有限过期和前缀一致性上失败
* 修复了 findById API 行为：在未找到时返回空值，而不是引发异常
* 修复了以下 bug：使用 CosmosPageRequest 时下一页未应用排序

### <a name="219-2019-12-26"></a>2.1.9 (2019-12-26)
#### <a name="new-features"></a>新增功能
* 添加了注释字段以启用/禁用自动创建集合
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 修复了 findById API 行为：在未找到时返回空值，而不是引发异常

### <a name="220-2019-10-21"></a>2.2.0 (2019-10-21)
#### <a name="new-features"></a>新增功能
* 完成 Reactive Cosmos Repository 支持
* Cosmos DB 请求诊断字符串和查询指标支持
* Cosmos DB SDK 版本更新为 3.3.1
* Spring Framework 版本升级为 5.2.0.RELEASE
* Spring Data Commons 版本升级为 2.2.0.RELEASE
* 添加了 findByIdAndPartitionKey、deleteByIdAndPartitionKey API
* 从 azure-doumentdb 中删除了依赖项
* 将 DocumentDb 更名为 Cosmos
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 修复了以下问题：pageSize 小于存储库中的总项时排序引发异常

### <a name="218-2019-10-18"></a>2.1.8 (2019-10-18)
#### <a name="new-features"></a>新增功能
* 弃用 Document DB API
* 添加了 findByIdAndPartitionKey、deleteByIdAndPartitionKey API。
* 添加了基于 _etag 的乐观锁定
* 对文档集合名称启用了 SPeL 表达式
* ObjectMapper 改进。
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="217-2019-10-18"></a>2.1.7 (2019-10-18)
#### <a name="new-features"></a>新增功能
* 添加了 Cosmos SDK v3 依赖项
* 添加了 Reactive Cosmos Repository
* 更新了 DocumentDbTemplate 的实现以使用 Cosmos SDK v3。
* Reactive Cosmos Repository 支持的其他配置更改。
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="212-2019-03-19"></a>2.1.2 (2019-03-19)
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 删除 applicationInsights 依赖项以解决以下问题
    * 依赖项污染的潜在风险
    * Java 11 不兼容
    * 避免对 CPU 和/或内存产生潜在的性能影响。

### <a name="207-2019-03-20"></a>2.0.7 (2019-03-20)
#### <a name="new-features"></a>新增功能
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 向后移植删除 applicationInsights 依赖项以解决以下问题
    * 依赖项污染的潜在风险
    * Java 11 不兼容
    * 避免对 CPU 和/或内存产生潜在的性能影响。

### <a name="211-2019-03-07"></a>2.1.1 (2019-03-07)
#### <a name="new-features"></a>新增功能
* 将主版本更新为 2.1.1
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="206-2019-03-07"></a>2.0.6 (2019-03-07)
#### <a name="new-features"></a>新增功能
* 忽略遥测中的所有异常
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="210-2018-12-17"></a>2.1.0 (2018-12-17)
#### <a name="new-features"></a>新增功能
* 将版本更新为 2.1.0 以解决问题
#### <a name="key-bug-fixes"></a>关键 Bug 修复

### <a name="205-2018-09-13"></a>2.0.5 (2018-09-13)
#### <a name="new-features"></a>新增功能
* 添加关键字 exists、startsWith
* 更新自述文件
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 修复以下问题：无法为 Entity 直接调用 self href
* 修复以下问题：如果未创建集合，findAll 将失败

### <a name="204-pre-release-2018-08-23"></a>2.0.4（预发行版）(2018-08-23)
#### <a name="new-features"></a>新增功能
* 将包从 documentdb 重命名为 cosmosdb，
* 新增查询方法关键字功能，支持 Sql API 中的 16 个关键字。
* 新增具有分页和排序的查询功能。
* 简化 spring-data-cosmosdb 的配置。
* 添加 deleteCollection 和 deleteAll API。

#### <a name="key-bug-fixes"></a>关键 Bug 修复
* Bug 修复和缺陷改进。

## <a name="faq"></a>常见问题
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>后续步骤
若要了解有关 Cosmos DB 的详细信息，请参阅 [Azure Cosmos DB](https://www.azure.cn/home/features/cosmos-db/) 服务页。

若要详细了解 Spring Framework，请参阅[项目主页](https://spring.io/projects/spring-framework)。

若要详细了解 Spring Boot，请参阅[项目主页](https://spring.io/projects/spring-boot)。

若要详细了解 Spring Data，请参阅[项目主页](https://spring.io/projects/spring-data)。

<!-- Update_Description: new article about sql api sdk java spring v2 -->
<!--NEW.date: 09/28/2020-->