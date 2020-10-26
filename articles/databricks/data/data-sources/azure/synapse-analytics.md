---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: Azure Synapse Analytics - Azure Databricks
description: 了解如何使用 Azure Databricks 在 Azure Synapse Analytics（以前称为 SQL 数据仓库）中读取和写入数据。
ms.openlocfilehash: f5f0829517a29c70f3eeb67894a0b50815b88737
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121827"
---
# <a name="azure-synapse-analytics"></a><a id="azure-synapse-analytics"> </a><a id="synapse-analytics"> </a>Azure Synapse Analytics

[Azure Synapse Analytics](/synapse-analytics/)（以前称为 SQL 数据仓库）是基于云的企业数据仓库，可利用大规模并行处理 (MPP) 对多达数 PB 的数据快速运行复杂的查询。 将 Azure 用作大数据解决方案的关键组件。 使用简单的 [PolyBase](/synapse-analytics/sql-data-warehouse/load-data-wideworldimportersdw) T-SQL 查询或 [COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql) 语句将大数据导入 Azure，然后利用 MPP 的能力运行高性能分析。 进行集成和分析时，数据仓库是企业获取见解能够依赖的唯一事实来源。

你可以使用 Azure Synapse 连接器（称为 Synapse 连接器）从 Azure Databricks 访问 Azure Synapse，该连接器是 Apache Spark 的数据源实现，它使用 [Azure Blob 存储](/storage/blobs/)、PolyBase 或 Azure Synapse 中的 `COPY` 语句在 Azure Databricks 群集和 Azure Synapse 实例之间有效地传输大量数据。

Azure Databricks 群集和 Azure Synapse 实例都访问公共 Blob 存储容器，以便在这两个系统之间交换数据。 在 Azure Databricks 中，Apache Spark 作业由 Azure Synapse 连接器触发，以便在 Blob 存储容器中读取和写入数据。 在 Azure Synapse 端，PolyBase 执行的数据加载和卸载操作由 Azure Synapse 连接器通过 JDBC 触发。 在 Databricks Runtime 7.0 及更高版本中，`COPY` 在默认情况下会通过 JDBC 由 Azure Synapse 连接器用来将数据加载到 Azure Synapse 中。

> [!NOTE]
>
> `COPY`
>
> * 为[公共预览版](../../../release-notes/release-types.md)。
> * 仅在 ADLS Gen2 上可用，后者提供[更好的性能](/sql-data-warehouse/upgrade-to-latest-generation)。 建议将数据库迁移到 ADLS Gen2。

Azure Synapse 连接器更适合 ETL 而不是交互式查询，因为每次执行查询都可以将大量数据提取到 Blob 存储中。 如果计划对同一 Azure Synapse 表执行多个查询，建议你以 Parquet 之类的格式保存提取的数据。

## <a name="requirements"></a>要求

用于 Azure Synapse 的[数据库主密钥](https://docs.microsoft.com/sql/relational-databases/security/encryption/create-a-database-master-key)。

## <a name="authentication"></a><a id="authentication"> </a><a id="azure-credentials"> </a>身份验证

Azure Synapse 连接器使用三种类型的网络连接：

* Spark 驱动程序到 Azure Synapse
* Spark 驱动程序和执行程序到 Azure 存储帐户
* Azure Synapse 到 Azure 存储帐户

```
                           ┌─────────┐
      ┌───────────────────>│ STORAGE │<──────────────────┐
      │ Storage acc key /  │ ACCOUNT │ Storage acc key / │
      │ Managed Service ID └─────────┘ OAuth 2.0         │
      │                         │                        │
      │                         │ Storage acc key /      │
      │                         │ OAuth 2.0              │
      v                         v                 ┌──────v────┐
┌──────────┐              ┌──────────┐            │┌──────────┴┐
│ Synapse  │              │  Spark   │            ││ Spark     │
│ Analytics│<────────────>│  Driver  │<───────────>| Executors │
└──────────┘  JDBC with   └──────────┘ Configured  └───────────┘
              username &               in Spark
              password
```

以下部分介绍每个连接的身份验证配置选项。

### <a name="spark-driver-to-azure-synapse"></a>Spark 驱动程序到 Azure Synapse

Spark 驱动程序使用 JDBC 以及用户名和密码连接到 Azure Synapse。
建议你使用 Azure 门户提供的连接字符串，该字符串使得通过 JDBC 连接在 Spark 驱动程序和 Azure Synapse 实例之间发送的所有数据都可以进行安全套接字层 (SSL) 加密。 若要验证是否已启用 SSL 加密，可以在连接字符串中搜索 `encrypt=true`。 为了使 Spark 驱动程序能够访问 Azure Synapse，建议你通过 Azure 门户在 Azure Synapse 服务器的防火墙窗格上将“允许访问 Azure 服务”设置为“打开”。 
此设置允许来自所有 Azure IP 地址和所有 Azure 子网的通信，使 Spark 驱动程序能够访问 Azure Synapse 实例。

### <a name="spark-driver-and-executors-to-azure-storage-account"></a>Spark 驱动程序和执行程序到 Azure 存储帐户

Azure 存储容器充当中介，用于在 Azure Synapse 中进行读取或写入操作时存储批量数据。 Spark 使用以下内置连接器之一连接到存储容器：[Azure Blob 存储](azure-storage.md#azure-storage)或 [Azure Data Lake Storage (ADLS) Gen2](azure-datalake-gen2.md#adls-gen2)。 因此，只有 `wasbs` 和 `abfss` 是受支持的 URI 方案。

用于设置此连接的凭据必须是存储帐户访问密钥和机密（Blob 和 ADLS Gen2）或 OAuth 2.0 令牌（仅 ADLS Gen2，请参阅[使用服务主体直接通过 OAuth 2.0 访问 Azure Data Lake Storage Gen2 帐户](azure-datalake-gen2.md#adls-gen2-oauth-2)）。
提供这些凭据的方式有两种：笔记本会话配置和全局 Hadoop 配置。
以下示例使用存储帐户访问密钥方法演示了这两种方式。 这同样适用于 OAuth 2.0 配置。

#### <a name="notebook-session-configuration-preferred"></a>笔记本会话配置（首选）

使用此方法时，将在与运行命令的笔记本关联的会话配置中设置帐户访问密钥。 此配置不影响附加到同一群集的其他笔记本。 `spark` 是笔记本中提供的 `SparkSession` 对象。

```python
spark.conf.set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")
```

#### <a name="global-hadoop-configuration"></a>全局 Hadoop 配置

此方法更新与 `SparkContext` 对象（由所有笔记本共享）相关联的全局 Hadoop 配置。

##### <a name="scala"></a>Scala

```scala
sc.hadoopConfiguration.set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")
```

##### <a name="python"></a>Python

`hadoopConfiguration` 并非在所有版本的 PySpark 中都公开。 尽管下面的命令依赖于某些 Spark 内部组件，但它应该适用于所有 PySpark 版本，将来不太可能停用或更改：

```python
sc._jsc.hadoopConfiguration().set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")
```

### <a name="azure-synapse-to-azure-storage-account"></a>Azure Synapse 到 Azure 存储帐户

在加载和卸载临时数据的过程中，Azure Synapse 还会连接到存储帐户。 若要在连接的 Azure Synapse 实例中设置存储帐户的凭据，可以将 `forwardSparkAzureStorageCredentials` 设置为 `true`，这样 Azure Synapse 连接器就会自动发现笔记本会话配置或全局 Hadoop 配置中设置的帐户访问密钥，并通过 JDBC 将存储帐户访问密钥转发到连接的 Azure Synapse 实例。
转发的存储访问密钥由 Azure Synapse 实例中的临时[数据库范围的凭据](https://docs.microsoft.com/sql/t-sql/statements/create-database-scoped-credential-transact-sql)表示。 Azure Synapse 连接器会在请求 Azure Synapse 加载或卸载数据之前创建数据库范围的凭据。 在加载或卸载操作完成后，连接器接着会删除数据库范围的凭据。

或者，如果你使用 ADLS Gen2 + OAuth 2.0 身份验证，或将 Azure Synapse 实例配置为具有一个托管服务标识（通常与 [VNet + 服务终结点设置](https://azure.microsoft.com/blog/general-availability-of-vnet-service-endpoints-for-azure-sql-data-warehouse/)配合使用），则必须将 `useAzureMSI` 设置为 `true`。 在这种情况下，连接器会为数据库范围的凭据指定 `IDENTITY = 'Managed Service Identity'`，并且不指定 `SECRET`。

## <a name="streaming-support"></a><a id="streaming-support"> </a><a id="streaming_support"> </a>流式处理支持

Azure Synapse 连接器为 Azure Synapse 提供高效且可缩放的结构化流式写入支持，以便提供对批量写入一致的用户体验，并使用 PolyBase 

<!--or `COPY`--> 在 Azure Databricks 群集和 Azure Synapse 实例之间进行大型数据传输。 与批量写入类似，流式处理主要用于 ETL，其延迟较高，因此在某些情况下可能不适合实时数据处理。

Azure Synapse 连接器支持用于记录追加和聚合的 `Append` 和 `Complete` 输出模式。 请参阅[结构化流式处理指南](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#output-modes)，详细了解输出模式和兼容性矩阵。

### <a name="fault-tolerance-semantics"></a>容错语义

默认情况下，Azure Synapse 流式处理提供端到端“恰好一次”保证，可确保将数据写入 Azure Synapse 表，方法是：将 DBFS 中的检查点位置、Azure Synapse 中的检查点表以及锁定机制组合使用，从而可靠地跟踪查询进度，以确保流式处理可以应对任何类型的故障、重试和查询重启。
也可为 Azure Synapse 流式处理选择限制较少的“至少一次”语义，方法是将 `spark.databricks.sqldw.streaming.exactlyOnce.enabled` 选项设置为 `false`，这样，如果在连接到 Azure Synapse 时出现间歇性故障，或者查询意外终止，则会进行数据复制。

## <a name="usage-batch"></a>用法（批处理）

可以在 Scala、Python、SQL 和 R 笔记本中通过数据源 API 使用此连接器。

### <a name="scala"></a>Scala

```scala
// Set up the Blob storage account access key in the notebook session conf.
spark.conf.set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")

// Get some data from an Azure Synapse table.
val df: DataFrame = spark.read
  .format("com.databricks.spark.sqldw")
  .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>")
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")
  .option("forwardSparkAzureStorageCredentials", "true")
  .option("dbTable", "my_table_in_dw")
  .load()

// Load data from an Azure Synapse query.
val df: DataFrame = spark.read
  .format("com.databricks.spark.sqldw")
  .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>")
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")
  .option("forwardSparkAzureStorageCredentials", "true")
  .option("query", "select x, count(*) as cnt from my_table_in_dw group by x")
  .load()

// Apply some transformations to the data, then use the
// Data Source API to write the data back to another table in Azure Synapse.

df.write
  .format("com.databricks.spark.sqldw")
  .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>")
  .option("forwardSparkAzureStorageCredentials", "true")
  .option("dbTable", "my_table_in_dw_copy")
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")
  .save()
```

### <a name="python"></a>Python

```python
# Set up the Blob storage account access key in the notebook session conf.
spark.conf.set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")

# Get some data from an Azure Synapse table.
df = spark.read \
  .format("com.databricks.spark.sqldw") \
  .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>") \
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>") \
  .option("forwardSparkAzureStorageCredentials", "true") \
  .option("dbTable", "my_table_in_dw") \
  .load()

# Load data from an Azure Synapse query.
df = spark.read \
  .format("com.databricks.spark.sqldw") \
  .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>") \
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>") \
  .option("forwardSparkAzureStorageCredentials", "true") \
  .option("query", "select x, count(*) as cnt from my_table_in_dw group by x") \
  .load()

# Apply some transformations to the data, then use the
# Data Source API to write the data back to another table in Azure Synapse.

df.write \
  .format("com.databricks.spark.sqldw") \
  .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>") \
  .option("forwardSparkAzureStorageCredentials", "true") \
  .option("dbTable", "my_table_in_dw_copy") \
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>") \
  .save()
```

### <a name="sql"></a>SQL

```sql
-- Set up the Blob storage account access key in the notebook session conf.
SET fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn=<your-storage-account-access-key>;

-- Read data using SQL.
CREATE TABLE my_table_in_spark_read
USING com.databricks.spark.sqldw
OPTIONS (
  url 'jdbc:sqlserver://<the-rest-of-the-connection-string>',
  forwardSparkAzureStorageCredentials 'true',
  dbTable 'my_table_in_dw',
  tempDir 'wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>'
);

-- Write data using SQL.
-- Create a new table, throwing an error if a table with the same name already exists:

CREATE TABLE my_table_in_spark_write
USING com.databricks.spark.sqldw
OPTIONS (
  url 'jdbc:sqlserver://<the-rest-of-the-connection-string>',
  forwardSparkAzureStorageCredentials 'true',
  dbTable 'my_table_in_dw_copy',
  tempDir 'wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>'
)
AS SELECT * FROM table_to_save_in_spark;
```

### <a name="r"></a>R

```r
# Load SparkR
library(SparkR)

# Set up the Blob storage account access key in the notebook session conf.
conf <- sparkR.callJMethod(sparkR.session(), "conf")
sparkR.callJMethod(conf, "set", "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn", "<your-storage-account-access-key>")

# Get some data from an Azure Synapse table.
df <- read.df(
   source = "com.databricks.spark.sqldw",
   url = "jdbc:sqlserver://<the-rest-of-the-connection-string>",
   forward_spark_azure_storage_credentials = "true",
   dbTable = "my_table_in_dw",
   tempDir = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")

# Load data from an Azure Synapse query.
df <- read.df(
   source = "com.databricks.spark.sqldw",
   url = "jdbc:sqlserver://<the-rest-of-the-connection-string>",
   forward_spark_azure_storage_credentials = "true",
   query = "select x, count(*) as cnt from my_table_in_dw group by x",
   tempDir = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")

# Apply some transformations to the data, then use the
# Data Source API to write the data back to another table in Azure Synapse.

write.df(
  df,
  source = "com.databricks.spark.sqldw",
  url = "jdbc:sqlserver://<the-rest-of-the-connection-string>",
  forward_spark_azure_storage_credentials = "true",
  dbTable = "my_table_in_dw_copy",
  tempDir = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")
```

## <a name="usage-streaming"></a>用法（流式处理）

可以在 Scala 和 Python 笔记本中使用结构化流式处理来写入数据。

### <a name="scala"></a>Scala

```scala
// Set up the Blob storage account access key in the notebook session conf.
spark.conf.set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")

// Prepare streaming source; this could be Kafka or a simple rate stream.
val df: DataFrame = spark.readStream
  .format("rate")
  .option("rowsPerSecond", "100000")
  .option("numPartitions", "16")
  .load()

// Apply some transformations to the data then use
// Structured Streaming API to continuously write the data to a table in Azure Synapse.

df.writeStream
  .format("com.databricks.spark.sqldw")
  .option("url", <azure-sqldw-jdbc-url>)
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>")
  .option("forwardSparkAzureStorageCredentials", "true")
  .option("dbTable", <table-name>)
  .option("checkpointLocation", "/tmp_checkpoint_location")
  .start()
```

### <a name="python"></a>Python

```python
# Set up the Blob storage account access key in the notebook session conf.
spark.conf.set(
  "fs.azure.account.key.<your-storage-account-name>.blob.core.chinacloudapi.cn",
  "<your-storage-account-access-key>")

# Prepare streaming source; this could be Kafka or a simple rate stream.
df = spark.readStream \
  .format("rate") \
  .option("rowsPerSecond", "100000") \
  .option("numPartitions", "16") \
  .load()

# Apply some transformations to the data then use
# Structured Streaming API to continuously write the data to a table in Azure Synapse.

df.writeStream \
  .format("com.databricks.spark.sqldw") \
  .option("url", <azure-sqldw-jdbc-url>) \
  .option("tempDir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.chinacloudapi.cn/<your-directory-name>") \
  .option("forwardSparkAzureStorageCredentials", "true") \
  .option("dbTable", <table-name>) \
  .option("checkpointLocation", "/tmp_checkpoint_location") \
  .start()
```

## <a name="configuration"></a>配置

此部分介绍如何配置连接器的写入语义、所需权限和其他配置参数。

### <a name="in-this-section"></a>本节内容：

* [写入语义](#write-semantics)
* [PolyBase 所需的 Azure Synapse 权限](#required-azure-synapse-permissions-for-polybase)
* [`COPY` 语句所需的 Azure Synapse 权限](#required-azure-synapse-permissions-for-the-copy-statement)
* [Parameters](#parameters)
* [将查询下推到 Azure Synapse 中](#query-pushdown-into-azure-synapse)
* [临时数据管理](#temporary-data-management)
* [临时对象管理](#temporary-object-management)
* [流式处理检查点表管理](#streaming-checkpoint-table-management)

### <a name="write-semantics"></a>写入语义

除 PolyBase 外，Azure Synapse 连接器还支持 `COPY` 语句。 `COPY` 语句提供了一种更方便的将数据加载到 Azure Synapse 中的方法，无需创建外部表。另外，此语句只需较少的权限即可加载数据，并且提高了性能，可以将数据以高吞吐量的方式引入到 Azure Synapse 中。

你可以使用以下配置来强制执行写入语义：

#### <a name="scala"></a>Scala

```scala
// Configure the write semantics for Azure Synapse connector in the notebook session conf.
spark.conf.set("spark.databricks.sqldw.writeSemantics", "<write-semantics>")
```

#### <a name="python"></a>Python

```python
# Configure the write semantics for Azure Synapse connector in the notebook session conf.
spark.conf.set("spark.databricks.sqldw.writeSemantics", "<write-semantics>")
```

#### <a name="sql"></a>SQL

```sql
-- Configure the write semantics for Azure Synapse connector in the notebook session conf.
SET spark.databricks.sqldw.writeSemantics=<write-semantics>;
```

#### <a name="r"></a>R

```r
# Load SparkR
library(SparkR)

# Configure the write semantics for Azure Synapse connector in the notebook session conf.
conf <- sparkR.callJMethod(sparkR.session(), "conf")
sparkR.callJMethod(conf, "set", "spark.databricks.sqldw.writeSemantics", "<write-semantics>")
```

其中，`<write-semantics>` 为：

* `polybase`（使用 PolyBase 将数据加载到 Azure Synapse 中）
* `copy`（在 Databricks Runtime 7.0 及更高版本中，使用 `COPY` 语句将数据加载到 Azure Synapse 中）
* 未指定（回退到默认值：对于 Databricks Runtime 7.0 及更高版本上的 ADLS Gen2，连接器会使用 `copy`，在其余情况下则使用 `polybase`）

### <a name="required-azure-synapse-permissions-for-polybase"></a><a id="dw-polybase-permissions"> </a><a id="required-azure-synapse-permissions-for-polybase"> </a>PolyBase 所需的 Azure Synapse 权限

使用 PolyBase 时，Azure Synapse 连接器要求 JDBC 连接用户有权在连接的 Azure Synapse 实例中运行以下命令：

* [CREATE DATABASE SCOPED CREDENTIAL](https://docs.microsoft.com/sql/t-sql/statements/create-database-scoped-credential-transact-sql)
* [CREATE EXTERNAL DATA SOURCE](https://docs.microsoft.com/sql/t-sql/statements/create-external-data-source-transact-sql)
* [CREATE EXTERNAL FILE FORMAT](https://docs.microsoft.com/sql/t-sql/statements/create-external-file-format-transact-sql)
* [CREATE EXTERNAL TABLE](https://docs.microsoft.com/sql/t-sql/statements/create-external-table-transact-sql)

连接器要求已存在一个用于指定的 Azure Synapse 实例的数据库主密钥，这是第一个命令的先决条件。 如果未满足该先决条件，可以使用 [CREATE MASTER KEY](https://docs.microsoft.com/sql/t-sql/statements/create-master-key-transact-sql) 命令创建一个密钥。

此外，若要读取通过 `dbTable` 设置的 Azure Synapse 表或在 `query` 中引用的表，JDBC 用户必须有权访问所需的 Azure Synapse 表。 若要将数据写回到通过 `dbTable` 设置的 Azure Synapse 表，JDBC 用户必须有权将数据写入此 Azure Synapse 表。

下表汇总了通过 PolyBase 执行的所有操作的权限：

| 操作       | 权限  |
|-----------------|--------------|
| 批量写入     | CONTROL      |
| 流式写入 | CONTROL      |
| 读取            | CONTROL      |

### <a name="required-azure-synapse-permissions-for-the-copy-statement"></a><a id="dw-copy-permissions"> </a><a id="required-azure-synapse-permissions-for-the-copy-statement"> </a>`COPY` 语句所需的 Azure Synapse 权限

> [!NOTE]
>
> 在 Databricks Runtime 7.0 及更高版本中可用。

使用 `COPY` 语句时，Azure Synapse 连接器要求 JDBC 连接用户有权在连接的 Azure Synapse 实例中运行以下命令：

* [COPY INTO](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql)

如果目标表不存在于 Azure Synapse 中，则需要运行以下命令以及上述命令的权限：

* [CREATE TABLE](https://docs.microsoft.com/sql/t-sql/statements/create-table-azure-sql-data-warehouse)

下表汇总了通过 `COPY` 进行批量写入和流式写入的权限：

| 操作       | 权限（插入到现有表中）       | 权限（插入到新表中）                                                               |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| 批量写入     | ADMINISTER DATABASE BULK OPERATIONS<br><br>INSERT | ADMINISTER DATABASE BULK OPERATIONS<br><br>INSERT<br><br>CREATE TABLE<br><br>ALTER ON SCHEMA :: dbo |
| 流式写入 | ADMINISTER DATABASE BULK OPERATIONS<br><br>INSERT | ADMINISTER DATABASE BULK OPERATIONS<br><br>INSERT<br><br>CREATE TABLE<br><br>ALTER ON SCHEMA :: dbo |

### <a name="parameters"></a>parameters

<!--There are docs for other options in https://github.com/databricks/universe/commit/58c664971b51eaacd1fd4c286fb292b8b7625143-->

Spark SQL 中提供的参数映射或 `OPTIONS` 支持以下设置：

| 参数                             | 必须                           | 默认                                                     | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|---------------------------------------|------------------------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dbTable`                             | 是，除非指定了 `query`   | 无默认值                                                  | 要在 Azure Synapse 中创建或读取的表。 将数据保存回 Azure Synapse 时，此参数是必需的。<br><br>你还可以使用 `{SCHEMA NAME}.{TABLE NAME}` 来访问采用给定架构的表。 如果未提供架构名称，则会使用与 JDBC 用户关联的默认架构。<br><br>先前支持的 `dbtable` 变体已弃用，在将来的版本中会被忽略。 请改用“混合大小写”名称。                                                                                                                                                                                                                                                                |
| `query`                               | 是，除非指定了 `dbTable` | 无默认值                                                  | 要从 Azure Synapse 中进行读取的查询。<br><br>对于在查询中引用的表，你还可以使用 `{SCHEMA NAME}.{TABLE NAME}` 来访问采用给定架构的表。 如果未提供架构名称，则会使用与 JDBC 用户关联的默认架构。                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `user`                                | 否                                 | 无默认值                                                  | Azure Synapse 用户名。 必须与 `password` 选项一起使用。 使用它的前提是未在 URL 中传递用户和密码。 同时传递这两项会导致错误。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `password`                            | 否                                 | 无默认值                                                  | Azure Synapse 密码。 必须与 `user` 选项一起使用。 使用它的前提是未在 URL 中传递用户和密码。 同时传递这两项会导致错误。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `url`                                 | 是                                | 无默认值                                                  | 一个 JDBC URL，其中的 `sqlserver` 设置为子协议。 建议使用 Azure 门户提供的连接字符串。 设置<br>强烈建议使用 `encrypt=true`，因为它允许对 JDBC 连接进行 SSL 加密。 如果单独设置了 `user` 和 `password`，则不需要在 URL 中包含它们。                                                                                                                                                                                                                                                                                                                                                                                  |
| `jdbcDriver`                          | 否                                 | 取决于 JDBC URL 的子协议                    | 要使用的 JDBC 驱动程序的类名。 此类必须位于类路径中。 在大多数情况下无需指定此选项，因为相应的驱动程序类名会由 JDBC URL 的子协议自动确定。<br><br>先前支持的 `jdbc_driver` 变体已弃用，在将来的版本中会被忽略。 请改用“混合大小写”名称。                                                                                                                                                                                                                                                                                                               |
| `tempDir`                             | 是                                | 无默认值                                                  | 一个 `wasbs` URI。 建议将专用 Blob 存储容器用于 Azure Synapse。<br><br>先前支持的 `tempdir` 变体已弃用，在将来的版本中会被忽略。 请改用“混合大小写”名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `tempFormat`                          | 否                                 | `PARQUET`                                                   | 将数据写入 Azure Synapse 时用来将临时文件保存到 blob 存储中的格式。 默认为 `PARQUET`；目前不允许其他值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `tempCompression`                     | 否                                 | `SNAPPY`                                                    | 由 Spark 和 Azure Synapse 用来进行临时编码/解码的压缩算法。 目前支持的值为 `UNCOMPRESSED`、`SNAPPY` 和 `GZIP`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `forwardSparkAzureStorageCredentials` | 否                                 | false                                                       | 如果此项为 `true`，库会自动发现 Spark 用来连接到 Blob 存储容器的凭据，并会通过 JDBC 将这些凭据转发到 Azure Synapse。 这些凭据作为 JDBC 查询的一部分发送。 因此，在使用此选项时，强烈建议你启用对 JDBC 连接进行 SSL 加密的功能。<br><br>Azure Synapse 连接器的当前版本要求将 `forwardSparkAzureStorageCredentials` 或 `useAzureMSI` 中的（恰好）一个显式设置为 `true`。<br><br>先前支持的 `forward_spark_azure_storage_credentials` 变体已弃用，在将来的版本中会被忽略。 请改用“混合大小写”名称。 |
| `useAzureMSI`                         | 否                                 | false                                                       | 如果此项为 `true`，则库会为它创建的数据库范围凭据指定 `IDENTITY = 'Managed Service Identity'` 并且不指定 `SECRET`。<br><br>Azure Synapse 连接器的当前版本要求将 `forwardSparkAzureStorageCredentials` 或 `useAzureMSI` 中的（恰好）一个显式设置为 `true`。                                                                                                                                                                                                                                                                                                                                                                                                  |
| `tableOptions`                        | 否                                 | `CLUSTERED COLUMNSTORE INDEX`, `DISTRIBUTION = ROUND_ROBIN` | 一个用于指定[表选项](https://docs.microsoft.com/sql/t-sql/statements/create-table-azure-sql-data-warehouse)的字符串。创建通过 `dbTable` 设置的 Azure Synapse 表时需要使用这些选项。 此字符串会以文本形式传递到针对 Azure Synapse 发出的 `CREATE TABLE` SQL 语句的 `WITH` 子句。<br><br>先前支持的 `table_options` 变体已弃用，在将来的版本中会被忽略。 请改用“混合大小写”名称。                                                                                                                                                                                                                                        |
| `preActions`                          | 否                                 | 无默认值（空字符串）                                   | 在将数据写入 Azure Synapse 实例之前要在 Azure Synapse 中执行的 SQL 命令的列表，其中各命令之间以 `;` 分隔。 这些 SQL 命令必须是 Azure Synapse 接受的有效命令。<br><br>如果这些命令中的任何一个失败，系统会将其视为错误，并且不会执行写入操作。                                                                                                                                                                                                                                                                                                                                                                                                    |
| `postActions`                         | 否                                 | 无默认值（空字符串）                                   | 在连接器成功将数据写入 Azure Synapse 实例后要在 Azure Synapse 中执行的 SQL 命令的列表，其中各命令之间以 `;` 分隔。 这些 SQL 命令必须是 Azure Synapse 接受的有效命令。<br><br>如果这些命令中的任何一个失败，系统会将其视为错误，并且，当数据成功写入 Azure Synapse 实例后，会出现异常。                                                                                                                                                                                                                                                                                                                  |
| `maxStrLength`                        | 否                                 | 256                                                         | Spark 中的 `StringType` 会映射到 Azure Synapse 中的 `NVARCHAR(maxStrLength)` 类型。 你可以使用 `maxStrLength` 为所有 `NVARCHAR(maxStrLength)` 类型列设置字符串长度，这些列位于 Azure Synapse 内名为<br>`dbTable` 的表中。<br><br>先前支持的 `maxstrlength` 变体已弃用，在将来的版本中会被忽略。 请改用“混合大小写”名称。                                                                                                                                                                                                                                                                                                             |
| `checkpointLocation`                  | 是                                | 无默认值                                                  | DBFS 上的位置，可供结构化流式处理用来写入元数据和检查点信息。 请参阅结构化流式处理编程指南中的 [Recovering from Failures with Checkpointing](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#recovering-from-failures-with-checkpointing)（使用检查点功能从故障中恢复）。                                                                                                                                                                                                                                                                                                                                                                                 |
| `numStreamingTempDirsToKeep`          | 否                                 | 0                                                           | 指示要保留多少（最新的）临时目录，以便定期清理流式处理中的微型批。 如果将此项设置为 `0`，则系统会在微型批提交后立即触发目录删除操作；如果将此项设置为其他值，则系统会保留所设定数量的最新微型批并删除其余目录。 使用 `-1` 可禁用定期清理。                                                                                                                                                                                                                                                                                                                                                                  |
| `applicationName`                     | 否                                 | `Databricks-User-Query`                                     | 每个查询的连接的标记。 如果未指定此项，或者值为空字符串，则会将标记的默认值添加到 JDBC URL。 默认值可防止 Azure DB 监视工具针对查询引发虚假 SQL 注入警报。                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `maxbinlength`                        | 否                                 | 无默认值                                                  | 控制 `BinaryType` 列的列长度。 此参数会转换为 `VARBINARY(maxbinlength)`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

> [!NOTE]
>
> * 仅当将数据从 Azure Databricks 写入 Azure Synapse 中的新表时，`tableOptions`、`preActions`、`postActions` 和 `maxStrLength` 才适用。
> * `checkpointLocation` 和 `numStreamingTempDirsToKeep` 仅适用于将数据从 Azure Databricks 流式写入到 Azure Synapse 中的新表。
> * 即使所有数据源选项名称不区分大小写，也建议你为了清楚起见，以“混合大小写”方式指定这些名称。

<!--TODO: Add a section about Data Type Mappings-->

### <a name="query-pushdown-into-azure-synapse"></a><a id="query-pushdown-into-azure-synapse"> </a><a id="sql-dw-query-pushdown"> </a>将查询下推到 Azure Synapse 中

Azure Synapse 连接器实施了一组将以下运算符下推到 Azure Synapse 中的优化规则：

* `Filter`
* `Project`
* `Limit`

`Project` 和 `Filter` 运算符支持以下表达式：

* 大多数布尔逻辑运算符
* 比较
* 基本算术运算
* 数值和字符串强制转换

对于 `Limit` 运算符，仅在未指定排序的情况下才支持下推。 例如： 。

`SELECT TOP(10) * FROM table`（而不是 `SELECT TOP(10) * FROM table ORDER BY col`）。

> [!NOTE]
>
> Azure Synapse 连接器不下推针对字符串、日期或时间戳进行运算的表达式。

默认启用通过 Azure Synapse 连接器构建的查询下推。
可以通过将 `spark.databricks.sqldw.pushdown` 设置为 `false` 来禁用它。

### <a name="temporary-data-management"></a><a id="temp-data-mgmt"> </a><a id="temporary-data-management"> </a>临时数据管理

Azure Synapse 连接器不会删除它在 Blob 存储容器中创建的临时文件。
因此，建议你定期删除用户提供的 `tempDir` 位置下的临时文件。

为了便于进行数据清理，Azure Synapse 连接器不会直接在 `tempDir` 下存储数据文件，而是创建如下格式的子目录：`<tempDir>/<yyyy-MM-dd>/<HH-mm-ss-SSS>/<randomUUID>/`。
可以通过设置定期作业（使用 Azure Databricks 的[作业](../../../jobs.md)功能或其他功能进行设置），以递归方式删除其创建时间早于给定阈值（例如 2 天）的任何子目录（假设 Spark 作业的运行时间不能超过该阈值）。

一个更简单的替代方法是，定期删除整个容器，然后使用同一名称创建一个新容器。
这要求你将专用容器用于 Azure Synapse 连接器生成的临时数据，并且你可以找到一个时间窗口，在该窗口中，你可以保证任何涉及连接器的查询均未在运行。

<!--commenting out because we don't talk about things that aren't true yet in doc .. note::-->

<!--The Azure Storage Team has announced plans to add support for `defining custom expiration policies on blobs <https://feedback.azure.com/forums/217298-storage/suggestions/2474308-provide-time-to-live-feature-for-blobs>`_-->

<!--in the future. Such a feature should eliminate the need for setting up cleanup jobs and thus considerably-->

<!--improve user experience.-->

### <a name="temporary-object-management"></a><a id="temp-object-mgmt"> </a><a id="temporary-object-management"> </a>临时对象管理

Azure Synapse 连接器在 Azure Databricks 群集和 Azure Synapse 实例之间自动进行数据传输。
为了从 Azure Synapse 表或查询中读取数据或将数据写入 Azure Synapse 表，Azure Synapse 连接器会创建临时对象，其中包括幕后的 `DATABASE SCOPED CREDENTIAL`、`EXTERNAL DATA SOURCE`、`EXTERNAL FILE FORMAT` 和 `EXTERNAL TABLE`。 这些对象只在相应 Spark 作业的整个持续时间内生存，此后会被自动删除。

当群集使用 Azure Synapse 连接器运行查询时，如果 Spark 驱动程序进程崩溃或被强制重启，或者群集被强制终止或重启，则可能不会删除临时对象。
为了便于识别并手动删除这些对象，Azure Synapse 连接器会使用以下格式的标记为在 Azure Synapse 实例中创建的所有中间临时对象的名称加上前缀：`tmp_<yyyy_MM_dd_HH_mm_ss_SSS>_<randomUUID>_`。

建议使用如下所示的查询定期查找泄漏的对象：

* `SELECT * FROM sys.database_scoped_credentials WHERE name LIKE 'tmp_databricks_%'`
* `SELECT * FROM sys.external_data_sources WHERE name LIKE 'tmp_databricks_%'`
* `SELECT * FROM sys.external_file_formats WHERE name LIKE 'tmp_databricks_%'`
* `SELECT * FROM sys.external_tables WHERE name LIKE 'tmp_databricks_%'`

### <a name="streaming-checkpoint-table-management"></a><a id="streaming-checkpoint-table-management"> </a><a id="streaming-checkpoint-table-mgmt"> </a>流式处理检查点表管理

Azure Synapse 连接器不会删除在新的流式查询启动时创建的流式处理检查点表。
此行为与 DBFS 上的 `checkpointLocation` 一致。 因此，对于将来不会运行的查询或已删除检查点位置的查询，建议你在删除 DBFS 上的检查点位置的同时定期删除检查点表。

默认情况下，所有检查点表的名称都为 `<prefix>_<query_id>`，其中 `<prefix>` 是一个可配置前缀（默认值为 `databricks_streaming_checkpoint`），`query_id` 是删除了 `_` 字符的流式查询 ID。 若要查找陈旧的或已删除的流式查询的所有检查点表，请运行以下查询：

```sql
SELECT * FROM sys.tables WHERE name LIKE 'databricks_streaming_checkpoint%'
```

可以通过 Spark SQL 配置选项 `spark.databricks.sqldw.streaming.exactlyOnce.checkpointTableNamePrefix` 来配置前缀。

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**我在使用 Azure Synapse 连接器时收到一个错误。如何判断此错误是来自 Azure Synapse 还是来自 Azure Databricks？**

为了帮助你调试错误，特定于 Azure Synapse 连接器的代码引发的任何异常都会包装在一个扩展 `SqlDWException` 特征的异常中。 异常还会进行以下区分：

* `SqlDWConnectorException` 表示 Azure Synapse 连接器引发的错误
* `SqlDWSideException` 表示连接的 Azure Synapse 实例引发的错误

**如果查询失败，并出现“在会话配置或全局 Hadoop 配置中找不到访问密钥”错误，该怎么办？**

此错误意味着，Azure Synapse 连接器在适用于 `tempDir` 中指定的存储帐户的笔记本会话配置或全局 Hadoop 配置中找不到存储帐户访问密钥。
有关如何正确配置存储帐户访问权限的示例，请参阅[用法（批处理）](#usage-batch)。 如果使用 Azure Synapse 连接器创建 Spark 表，则仍然必须提供存储帐户访问凭据才能在 Spark 表中进行读取或写入操作。

**能否使用共享访问签名 (SAS) 访问通过 `tempDir` 指定的 Blob 存储容器？**

Azure Synapse 不支持[使用 SAS 访问 Blob 存储](https://docs.microsoft.com/sql/t-sql/statements/create-database-scoped-credential-transact-sql#b-creating-a-database-scoped-credential-for-a-shared-access-signature)。 因此，Azure Synapse 连接器不支持使用 [SAS](/storage/common/storage-dotnet-shared-access-signature-part-1) 来访问通过 `tempDir` 指定的 Blob 存储容器。

**我通过将 Azure Synapse 连接器与 `dbTable` 选项配合使用创建了一个 Spark 表，并向此 Spark 表写入了一些数据，然后删除了此 Spark 表。在 Azure Synapse 端创建的表是否会被删除？**

否。 Azure Synapse 被视为外部数据源。 删除 Spark 表时，不会删除其名称通过 `dbTable` 进行设置的 Azure Synapse 表。

**将数据帧写入 Azure Synapse 时，为什么需要使用 `.option("dbTable", tableName).save()` 而不是直接使用 `.saveAsTable(tableName)`？**

这是因为我们想要使以下区分非常清晰：`.option("dbTable", tableName)` 是指数据库（即 Azure Synapse）表，而 `.saveAsTable(tableName)` 是指 Spark 表。 事实上，你甚至可以将二者结合在一起：`df.write. ... .option("dbTable", tableNameDW).saveAsTable(tableNameSpark)`，这样就会在 Azure Synapse 中创建一个名为 `tableNameDW` 的表，并会在 Spark 中创建一个名为 `tableNameSpark` 且受 Azure Synapse 表支持的外部表。

> [!WARNING]
>
> 请注意 `.save()` 和 `.saveAsTable()` 之间的以下差异：
>
> * 对于 `df.write. ... .option("dbTable", tableNameDW).mode(writeMode).save()`，`writeMode` 会按预期方式作用于 Azure Synapse 表。
> * 对于 `df.write. ... .option("dbTable", tableNameDW).mode(writeMode).saveAsTable(tableNameSpark)`，`writeMode` 会作用于 Spark 表，而 `tableNameDW` 则会被以无提示方式覆盖（如果它已存在于 Azure Synapse 中）。
>
> 此行为与将数据写入任何其他数据源没有什么不同。 这只是 Spark [DataFrameWriter API](https://github.com/apache/spark/blob/v2.3.0/sql/core/src/main/scala/org/apache/spark/sql/DataFrameWriter.scala) 的注意事项。

<!--Mention that if users hit the following error in Azure Synapse, use int instead of tiny int: Caused by: com.microsoft.sqlserver.jdbc.SQLServerException: Query aborted-- the maximum reject threshold (0 rows) was reached while reading from an external source: 1 rows rejected out of total 1 rows processed. Column ordinal: 1, Expected data type: TINYINT, Offending value: Value: 200 (Column Conversion Error), Error: Arithmetic overflow error converting tinyint to data type TINYINT.-->