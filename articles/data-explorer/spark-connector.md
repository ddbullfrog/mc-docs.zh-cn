---
title: 使用适用于 Apache Spark 的 Azure 数据资源管理器连接器在 Azure 数据资源管理器与 Spark 群集之间移动数据。
description: 本主题介绍如何在 Azure 数据资源管理器与 Apache Spark 群集之间移动数据。
author: orspod
ms.author: v-tawe
ms.reviewer: maraheja
ms.service: data-explorer
ms.topic: how-to
origin.date: 07/29/2020
ms.date: 09/24/2020
ms.openlocfilehash: 4856cd84a9da0fc40921fbc0e56dd9d0b005fc59
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146495"
---
# <a name="azure-data-explorer-connector-for-apache-spark"></a>适用于 Apache Spark 的 Azure 数据资源管理器连接器

[Apache Spark](https://spark.apache.org/) 是用于大规模数据处理的统一分析引擎。 Azure 数据资源管理器是一个快速、完全托管的数据分析服务，可用于实时分析大量数据。 

适用于 Spark 的 Azure 数据资源管理器连接器是可在任何 Spark 群集上运行的[开源项目](https://github.com/Azure/azure-kusto-spark)。 它实现了用于跨 Azure 数据资源管理器和 Spark 群集移动数据的数据源和数据接收器。 使用 Azure 数据资源管理器和 Apache Spark，可以构建面向数据驱动型方案的可缩放快速应用程序。 例如，机器学习 (ML)、提取-转换-加载 (ETL) 和 Log Analytics。 有了此连接器，Azure 数据资源管理器变成了标准 Spark 源和接收器操作（例如写入、读取和 writeStream）的有效数据存储。

你可以采用批处理模式或流式处理模式向 Azure 数据资源管理器进行写入。 从 Azure 数据资源管理器中读取支持列删除和谓词下推，这可在 Azure 数据资源管理器中筛选数据，从而减少传输的数据量。

本主题介绍了如何安装和配置 Azure 数据资源管理器 Spark 连接器，以及如何在 Azure 数据资源管理器与 Apache Spark 群集之间移动数据。

> [!NOTE]
> 尽管下面的某些示例提到了 [Azure Databricks](https://docs.azuredatabricks.net/) Spark 群集，但 Azure 数据资源管理器 Spark 连接器并不直接依赖于 Databricks 或任何其他 Spark 分发版。

## <a name="prerequisites"></a>先决条件

* [创建 Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md) 
* 创建 Spark 群集
* 安装 Azure 数据资源管理器连接器库：
    * [Spark 2.4、Scala 2.11](https://github.com/Azure/azure-kusto-spark/releases) 的预生成库 
    * [Maven 存储库](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/spark-kusto-connector)
* 已安装 [Maven 3.x](https://maven.apache.org/download.cgi)

> [!TIP]
> 也支持版本 2.3.x，不过，这可能需要在 pom.xml 依赖项中进行某些更改。

## <a name="how-to-build-the-spark-connector"></a>如何生成 Spark 连接器

> [!NOTE]
> 此步骤是可选的。 如果使用的是预生成库，请转到 [Spark 群集设置](#spark-cluster-setup)。

### <a name="build-prerequisites"></a>生成先决条件

1. 安装[依赖项](https://github.com/Azure/azure-kusto-spark#dependencies)中列出的库，包括以下 [Kusto Java SDK](kusto/api/java/kusto-java-client-library.md) 库：
    * [Kusto 数据客户端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-data)
    * [Kusto 引入客户端](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-ingest)

1. 参考[此源](https://github.com/Azure/azure-kusto-spark)来生成 Spark 连接器。

1. 对于使用 Maven 项目定义的 Scala/Java 应用程序，请将应用程序链接到以下项目（在最新版本中可能不同）：
    
    ```Maven
       <dependency>
         <groupId>com.microsoft.azure</groupId>
         <artifactId>spark-kusto-connector</artifactId>
         <version>1.1.0</version>
       </dependency>
    ```

### <a name="build-commands"></a>生成命令

生成 jar 并运行所有测试：

```
mvn clean package
```

生成 jar，运行所有测试，并将 jar 安装到本地 Maven 存储库：

```
mvn clean install
```

有关详细信息，请参阅[连接器用法](https://github.com/Azure/azure-kusto-spark#usage)。

## <a name="spark-cluster-setup"></a>Spark 群集设置

> [!NOTE]
> 建议使用最新的 Azure 数据资源管理器 Spark 连接器版本执行以下步骤。

1. 使用 Spark 2.4.4 和 Scala 2.11，基于 Azure Databricks 群集配置以下 Spark 群集设置：

    ![Databricks 群集设置](media/spark-connector/databricks-cluster.png)
    
1. 从 Maven 安装最新 spark-kusto-connector 库：
    
    ![导入库](media/spark-connector/db-libraries-view.png) ![选择 Spark-Kusto-Connector](media/spark-connector/db-dependencies.png)

1. 验证是否已安装所有必需的库：

    ![验证是否已安装库](media/spark-connector/db-libraries-view.png)

1. 对于使用 JAR 文件的安装，请验证是否安装了其他依赖项：

    ![添加依赖项](media/spark-connector/db-not-maven.png)

## <a name="authentication"></a>身份验证

使用 Azure 数据资源管理器 Spark 连接器，可以使用下列方法之一通过 Azure Active Directory (Azure AD) 进行身份验证：
* [Azure AD 应用程序](#azure-ad-application-authentication)
* [Azure AD 访问令牌](https://github.com/Azure/azure-kusto-spark/blob/master/docs/Authentication.md#direct-authentication-with-access-token)
* [设备身份验证](https://github.com/Azure/azure-kusto-spark/blob/master/docs/Authentication.md#device-authentication)（适用于非生产方案）
* [Azure Key Vault](https://github.com/Azure/azure-kusto-spark/blob/master/docs/Authentication.md#key-vault) 若要访问 Key Vault 资源，请安装 azure-keyvault 包并提供应用程序凭据。

### <a name="azure-ad-application-authentication"></a>Azure AD 应用程序身份验证

Azure AD 应用程序身份验证是最简单且最常用的身份验证方法，建议将其用于 Azure 数据资源管理器 Spark 连接器。

|属性  |选项字符串  |说明  |
|---------|---------|---------|
|**KUSTO_AAD_APP_ID**     |kustoAadAppId     |   Azure AD 应用程序（客户端）标识符。      |
|**KUSTO_AAD_AUTHORITY_ID**     |kustoAadAuthorityID     |  Azure AD 身份验证颁发机构。 Azure AD 目录（租户）ID。        |
|**KUSTO_AAD_APP_SECRET**    |kustoAadAppSecret     |    客户端的 Azure AD 应用程序密钥。     |

> [!NOTE]
> 较旧的 API 版本（低于 2.0.0）有以下命名：“kustoAADClientID”、“kustoClientAADClientPassword”、“kustoAADAuthorityID”

### <a name="azure-data-explorer-privileges"></a>Azure 数据资源管理器权限

必须在 Azure 数据资源管理器群集上授予以下权限：

* 对于读取操作（数据源），Azure AD 标识必须对目标数据库拥有“查看者”特权，或者对目标表拥有“管理员”特权。  
* 对于写入操作（数据接收器），Azure AD 标识必须对目标数据库拥有“引入者”特权。  此外，它必须对目标数据库拥有“用户”特权，这样才能创建新表。  如果目标表已存在，则必须配置对目标表的“管理员”权限。 
 
有关 Azure 数据资源管理器主体角色的详细信息，请参阅[基于角色的授权](kusto/management/access-control/role-based-authorization.md)。 有关如何管理安全角色，请参阅[安全角色管理](kusto/management/security-roles.md)。

## <a name="spark-sink-writing-to-azure-data-explorer"></a>Spark 接收器：写入 Azure 数据资源管理器

1. 设置接收器参数：

     ```scala
    val KustoSparkTestAppId = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppId")
    val KustoSparkTestAppKey = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppKey")
 
    val appId = KustoSparkTestAppId
    val appKey = KustoSparkTestAppKey
    val authorityId = "72f988bf-86f1-41af-91ab-2d7cd011db47" // Optional - defaults to microsoft.com
    val cluster = "Sparktest.chinanorth2"
    val database = "TestDb"
    val table = "StringAndIntTable"
    ```

1. 将 Spark 数据帧分批写入 Azure 数据资源管理器群集：

    ```scala
    import com.microsoft.kusto.spark.datasink.KustoSinkOptions
    import org.apache.spark.sql.{SaveMode, SparkSession}

    df.write
      .format("com.microsoft.kusto.spark.datasource")
      .option(KustoSinkOptions.KUSTO_CLUSTER, cluster)
      .option(KustoSinkOptions.KUSTO_DATABASE, database)
      .option(KustoSinkOptions.KUSTO_TABLE, "Demo3_spark")
      .option(KustoSinkOptions.KUSTO_AAD_APP_ID, appId)
      .option(KustoSinkOptions.KUSTO_AAD_APP_SECRET, appKey)
      .option(KustoSinkOptions.KUSTO_AAD_AUTHORITY_ID, authorityId)
      .option(KustoSinkOptions.KUSTO_TABLE_CREATE_OPTIONS, "CreateIfNotExist")
      .mode(SaveMode.Append)
      .save()  
    ```
    
   或者使用简化的语法：
   
    ```scala
         import com.microsoft.kusto.spark.datasink.SparkIngestionProperties
         import com.microsoft.kusto.spark.sql.extension.SparkExtension._
         
         val sparkIngestionProperties = Some(new SparkIngestionProperties()) // Optional, use None if not needed
         df.write.kusto(cluster, database, table, conf, sparkIngestionProperties)
    ```
   
1. 写入流数据：

    ```scala    
    import org.apache.spark.sql.streaming.Trigger
    import java.util.concurrent.TimeUnit
    import java.util.concurrent.TimeUnit
    import org.apache.spark.sql.streaming.Trigger

    // Set up a checkpoint and disable codeGen. 
    spark.conf.set("spark.sql.streaming.checkpointLocation", "/FileStore/temp/checkpoint")
        
    // Write to a Kusto table from a streaming source
    val kustoQ = df
          .writeStream
          .format("com.microsoft.kusto.spark.datasink.KustoSinkProvider")
          .options(conf) 
          .option(KustoSinkOptions.KUSTO_WRITE_ENABLE_ASYNC, "true") // Optional, better for streaming, harder to handle errors
          .trigger(Trigger.ProcessingTime(TimeUnit.SECONDS.toMillis(10))) // Sync this with the ingestionBatching policy of the database
          .start()
    ```

## <a name="spark-source-reading-from-azure-data-explorer"></a>Spark 源：从 Azure 数据资源管理器读取数据

1. 读取[少量数据](kusto/concepts/querylimits.md)时，可以定义数据查询：

    ```scala
    import com.microsoft.kusto.spark.datasource.KustoSourceOptions
    import org.apache.spark.SparkConf
    import org.apache.spark.sql._
    import com.microsoft.azure.kusto.data.ClientRequestProperties

    val query = s"$table | where (ColB % 1000 == 0) | distinct ColA"
    val conf: Map[String, String] = Map(
          KustoSourceOptions.KUSTO_AAD_APP_ID -> appId,
          KustoSourceOptions.KUSTO_AAD_APP_SECRET -> appKey
        )

    val df = spark.read.format("com.microsoft.kusto.spark.datasource").
      options(conf).
      option(KustoSourceOptions.KUSTO_QUERY, query).
      option(KustoSourceOptions.KUSTO_DATABASE, database).
      option(KustoSourceOptions.KUSTO_CLUSTER, cluster).
      load()

    // Simplified syntax flavor
    import com.microsoft.kusto.spark.sql.extension.SparkExtension._
    
    val cpr: Option[ClientRequestProperties] = None // Optional
    val df2 = spark.read.kusto(cluster, database, query, conf, cpr)
    display(df2)
    ```

1. 可选：如果**你**提供暂时性 blob 存储（而不是 Azure 数据资源管理器），则由调用方负责创建 blob。 这包括预配存储、轮换访问密钥以及删除暂时性项目。 
    KustoBlobStorageUtils 模块包含用于以下用途的帮助程序函数：基于帐户和容器坐标以及帐户凭据（或基于具有写入、读取和列出权限的完整 SAS URL）删除 blob。 当不再需要相应的 RDD 时，每个事务会将暂时性 blob 项目存储在一个单独的目录中。 此目录是作为 Spark 驱动程序节点上报告的读取事务信息日志的一部分捕获的。

    ```scala
    // Use either container/account-key/account name, or container SaS
    val container = dbutils.secrets.get(scope = "KustoDemos", key = "blobContainer")
    val storageAccountKey = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountKey")
    val storageAccountName = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountName")
    // val storageSas = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageSasUrl")
    ```

    在上面的示例中，无法使用连接器接口访问 Key Vault；使用了一种更简单的方法，即使用 Databricks 机密。

1. 从 Azure 数据资源管理器进行读取。

    * 如果**你**提供暂时性 blob 存储，请如下所示从 Azure 数据资源管理器进行读取：

        ```scala
         val conf3 = Map(
              KustoSourceOptions.KUSTO_AAD_APP_ID -> appId,
              KustoSourceOptions.KUSTO_AAD_APP_SECRET -> appKey
              KustoSourceOptions.KUSTO_BLOB_STORAGE_SAS_URL -> storageSas)
        val df2 = spark.read.kusto(cluster, database, "ReallyBigTable", conf3)
        
        val dfFiltered = df2
          .where(df2.col("ColA").startsWith("row-2"))
          .filter("ColB > 12")
          .filter("ColB <= 21")
          .select("ColA")
        
        display(dfFiltered)
        ```

    * 如果 **Azure 数据资源管理器**提供暂时性 blob 存储，请如下所示从 Azure 数据资源管理器进行读取：
    
        ```scala
        val dfFiltered = df2
          .where(df2.col("ColA").startsWith("row-2"))
          .filter("ColB > 12")
          .filter("ColB <= 21")
          .select("ColA")
        
        display(dfFiltered)
        ```

## <a name="next-steps"></a>后续步骤

* 详细了解 [Azure 数据资源管理器 Spark 连接器](https://github.com/Azure/azure-kusto-spark/tree/master/docs)
* [Java 和 Python 的示例代码](https://github.com/Azure/azure-kusto-spark/tree/master/samples/src/main)
