---
title: 用于 SQL API 的 Spring Data Azure Cosmos DB v3 的发行说明和资源
description: 了解有关用于 SQL API 的 Spring Data Azure Cosmos DB v3 的所有信息，包括发行日期、停用日期和 Azure Cosmos DB SQL Async Java SDK 各版本之间所做的更改。
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
origin.date: 08/18/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: devx-track-java
ms.openlocfilehash: 739fc60f57dcd34ae60e1ba8f92340a1da961b29
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91247003"
---
<!--Verified successfully-->
# <a name="spring-data-azure-cosmos-db-v3-for-core-sql-api-release-notes-and-resources"></a>用于 Core (SQL) API 的 Spring Data Azure Cosmos DB v3：发行说明和资源
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

通过用于 Core (SQL) 的 Spring Data Azure Cosmos DB v3，开发人员可以在 Spring 应用程序中使用 Azure Cosmos DB。 Spring Data Azure Cosmos DB 公开 Spring Data 接口，以便操作数据库和集合、使用文档和发出查询。 同一 Maven 项目中同时支持 Sync 和 Async (Reactive) API。 

Spring Data Azure Cosmos DB 依赖于 Spring Data 框架。 Azure Cosmos DB SDK 团队发布了适用于 Spring Data v2.2 和 v2.3 的 Maven 项目。

[Spring Framework](https://spring.io/projects/spring-framework) 是一种简化 Java 应用程序开发的编程和配置模型。 为引用组织的网站，Spring 使用依赖项注入来简化应用程序的“管道”。 由于生成和测试应用程序变得更加简单，因此许多开发人员喜欢 Spring。 [Spring Boot](https://spring.io/projects/spring-boot) 重视 Web 应用程序和微服务的开发，扩展了这种处理管道的理念。 [Spring Data](https://spring.io/projects/spring-data) 是一种编程模型和框架，用于从 Spring 或 Spring Boot 应用程序的上下文中访问数据存储（如 Azure Cosmos DB）。 

可在 [Azure Spring Cloud](https://www.azure.cn/home/features/spring-cloud/) 应用程序中使用 Spring Data Azure Cosmos DB。

<!--MOONCAKE CORRECT ON THE LINK-->

> [!IMPORTANT]  
> 这些发行说明适用于 Spring Data Azure Cosmos DB v3。 可以在[此处](sql-api-sdk-java-spring-v2.md)找到 v2 发行说明。 
>
> Spring Data Azure Cosmos DB 仅支持 SQL API。
>
> 以下指南支持其他 Azure Cosmos DB API 上的 Spring Data：
> * [将适用于 Apache Cassandra 的 Spring Data 用于 Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [将 Spring Data MongoDB 用于 Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [将 Spring Data Gremlin 用于 Azure Cosmos DB](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>

<!--MOONCAKE CORRECT ON THE LINK-->

## <a name="start-here"></a>从此处开始

# <a name="explore"></a>[探索](#tab/explore)

<img src="media/sql-api-sdk-java-spring-v3/up-arrow.png" alt="explore the tabs above" width="80"/>

### <a name="navigate-the-tabs-above-for-basic-spring-data-azure-cosmos-db-samples"></a>浏览上面的选项卡，以获取基本 Spring Data Azure Cosmos DB 示例。

# <a name="pomxml"></a>[pom.xml](#tab/pom)

### <a name="configure-dependencies"></a>配置依赖项

系统提供了两个 Spring Data Azure Cosmos DB v3 Maven 项目。

依赖于 Spring Data 框架 v2.2 的项目：
```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-2-2-cosmos</artifactId>
    <version>latest</version>
</dependency>
```

依赖于 Spring Data 框架 v2.3 的项目：
```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-2-3-cosmos</artifactId>
    <version>latest</version>
</dependency>
```

# <a name="connect"></a>[“连接”](#tab/connect)

### <a name="connect"></a>连接

指定 Azure Cosmos DB 帐户和容器详细信息。 Spring Data Azure Cosmos DB 会自动创建客户端并连接到容器。

[application.properties](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-2-cosmos-java-getting-started/src/main/resources/application.properties)：
```
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

# <a name="doc-ops"></a>[文档操作](#tab/docs)

### <a name="document-operations"></a>文档操作

[创建](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java)：

<!--MOONCAKE CUSTOMIZATION-->

```java
logger.info("Saving user : {}", testUser1);
userRepository.save(testUser1);
```


[删除](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java)：

```java
userRepository.deleteAll();
```

<!--MOONCAKE CUSTOMIZATION-->

# <a name="query"></a>[查询](#tab/queries)

### <a name="query"></a>查询

[查询](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-2-3-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java)：

```javaa
Flux<User> users = reactiveUserRepository.findByFirstName("testFirstName");
users.map(u -> {
    logger.info("user is : {}", u);
    return u;
}).subscribe();
```

---

## <a name="helpful-content"></a>帮助性内容

| Content | Spring Data 框架 v2.2 | Spring Data 框架 v2.3 |
|---|---|
| **SDK 下载** | [Maven](https://mvnrepository.com/artifact/com.azure/azure-spring-data-2-2-cosmos) | [Maven](https://mvnrepository.com/artifact/com.azure/azure-spring-data-2-3-cosmos) |
|**参与 SDK** | [GitHub 上的 Spring Data Azure Cosmos DB 存储库](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-2-2-cosmos) | [GitHub 上的 Spring Data Azure Cosmos DB 存储库](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-2-3-cosmos) | 
|**教程**| [GitHub 上的 Spring Data Azure Cosmos DB 教程](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/tree/main/azure-spring-data-2-2-cosmos-java-getting-started) | [GitHub 上的 Spring Data Azure Cosmos DB 教程](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/tree/main/azure-spring-data-2-3-cosmos-java-getting-started) |

## <a name="release-history"></a>版本历史记录

### <a name="300-beta2-unreleased"></a>3.0.0-beta.2（未发布）

### <a name="300-beta1-2020-08-17"></a>3.0.0-beta.1 (2020-08-17)
#### <a name="new-features"></a>新增功能
* 组 ID 更新为 `com.azure`。
* 项目 ID 更新为 `azure-spring-data-2-3-cosmos`。
* azure-cosmos SDK 依赖项更新为 `4.3.2-beta.2`。
* 支持审核实体 - 自动管理 createdBy、createdDate、lastModifiedBy 和 lastModifiedDate 注释字段。
* `@GeneratedValue` 注释支持为 `String` 类型的 ID 字段自动生成 ID。
* 多数据库配置支持具有多个数据库的单个 cosmos 帐户和具有多个数据库的多个 cosmos 帐户。
* 支持任何字符串字段上的 `@Version` 注释。
* 同步 API 返回类型更新为 `Iterable` 类型，而非 `List`。
* 将 `CosmosClientBuilder` 从 Cosmos SDK 作为 Spring Bean 公开给 `@Configuration` 类。
* 更新了 `CosmosConfig`，以包含查询指标和响应诊断处理器实现。
* 支持为单个结果查询返回 `Optional` 数据类型。
#### <a name="renames"></a>重命名
* `CosmosDbFactory` 重命名为 `CosmosFactory`。
* `CosmosDBConfig` 重命名为 `CosmosConfig`。
* `CosmosDBAccessException` 重命名为 `CosmosAccessException`。
* `Document` 注释重命名为 `Container` 注释。
* `DocumentIndexingPolicy` 注释重命名为 `CosmosIndexingPolicy` 注释。
* `DocumentQuery` 重命名为 `CosmosQuery`。
* application.properties 标志 `populateQueryMetrics` 重命名为 `queryMetricsEnabled`。
#### <a name="key-bug-fixes"></a>关键 Bug 修复
* 将诊断日志记录任务调度给 `Parallel` 线程，以避免阻塞 Netty I/O 线程。
* 修复了对删除操作的乐观锁定。
* 修复了对 `IN` 子句查询进行转义时出现的问题。
* 通过允许为 `@Id` 使用 `long` 数据类型修复了问题。
* 通过允许将 `boolean`、`long`、`int`、`double` 作为 `@PartitionKey` 注释的数据类型修复了问题。
* 为忽略大小写查询修复了 `IgnoreCase` & `AllIgnoreCase` 关键字。
* 删除了自动创建容器时的默认请求单位值 4000。

## <a name="faq"></a>常见问题解答
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>后续步骤
若要了解有关 Cosmos DB 的详细信息，请参阅 [Azure Cosmos DB](https://www.azure.cn/home/features/cosmos-db/) 服务页。

若要详细了解 Spring Framework，请参阅[项目主页](https://spring.io/projects/spring-framework)。

若要详细了解 Spring Boot，请参阅[项目主页](https://spring.io/projects/spring-boot)。

若要详细了解 Spring Data，请参阅[项目主页](https://spring.io/projects/spring-data)。

<!-- Update_Description: new article about sql api sdk java spring v3 -->
<!--NEW.date: 09/28/2020-->