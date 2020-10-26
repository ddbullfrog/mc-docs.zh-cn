---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 使用 JDBC 的 SQL 数据库 - Azure Databricks
description: 了解如何使用 Azure Databricks 读取数据并将数据写入 Microsoft SQL Server、MariaDB、mySQL 和其他与 JDBC 兼容的数据库。
ms.openlocfilehash: a5bf88332dd2a647ca96e33612018b7957c74f9e
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121936"
---
# <a name="sql-databases-using-jdbc"></a><a id="sql-databases-using-jdbc"> </a><a id="sql-jdbc"> </a>使用 JDBC 的 SQL 数据库

Databricks Runtime 包含 [Microsoft SQL Server](https://docs.microsoft.com/sql/sql-server/sql-server-technical-documentation) 和 [Azure SQL 数据库](/sql-database/)的 JDBC 驱动程序。 若要查看 Databricks Runtime 中包含的 JDBC 库的完整列表，请参阅 [Databricks Runtime 发行说明](../../release-notes/runtime/index.md)。

本文介绍如何使用数据帧 API 连接到使用 JDBC 的 SQL 数据库，以及如何控制通过 JDBC 接口进行的读取操作的并行度。 本文提供了使用 Scala API 的详细示例，并在末尾提供了 Python 和 Spark SQL 的简略示例。 若要查看用于连接到使用 JDBC 的 SQL 数据库的所有受支持的参数，请参阅[使用 JDBC 连接到其他数据库](https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html)。

> [!NOTE]
>
> 还有一种连接到 SQL Server 和 Azure SQL 数据库的方法是使用 [Apache Spark 连接器](sql-databases-azure.md#azure-db)。 它可提供更快的批量插入，让你能够使用 Azure Active Directory 标识进行连接。

> [!IMPORTANT]
>
> 本文中的示例不包括 JDBC URL 中的用户名和密码。 而是预设你按照[机密管理](../../security/secrets/index.md#secrets-user-guide)用户指南中的内容将自己的数据库凭据存储为机密，然后在笔记本中利用它们填充 `java.util.Properties` 对象中的凭据。 例如： 。
>
> ```scala
> val jdbcUsername = dbutils.secrets.get(scope = "jdbc", key = "username")
> val jdbcPassword = dbutils.secrets.get(scope = "jdbc", key = "password")
> ```
>
> 若要查看机密管理的完整示例，请参阅[机密工作流示例](../../security/secrets/example-secret-workflow.md)。

## <a name="establish-connectivity-to-sql-server"></a>建立与 SQL Server 的连接

此示例使用 SQL Server 的 JDBC 驱动程序对其进行查询。

### <a name="step-1-check-that-the-jdbc-driver-is-available"></a>步骤 1：检查 JDBC 驱动程序是否可用

```scala
Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver")
```

### <a name="step-2-create-the-jdbc-url"></a>步骤 2：创建 JDBC URL

```scala
val jdbcHostname = "<hostname>"
val jdbcPort = 1433
val jdbcDatabase = "<database>"

// Create the JDBC URL without passing in the user and password parameters.
val jdbcUrl = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase}"

// Create a Properties() object to hold the parameters.
import java.util.Properties
val connectionProperties = new Properties()

connectionProperties.put("user", s"${jdbcUsername}")
connectionProperties.put("password", s"${jdbcPassword}")
```

### <a name="step-3-check-connectivity-to-the-sqlserver-database"></a>步骤 3：检查与 SQLServer 数据库之间的连接

```scala
val driverClass = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
connectionProperties.setProperty("Driver", driverClass)
```

## <a name="read-data-from-jdbc"></a>从 JDBC 读取数据

本部分从数据库表加载数据。 这会使用单个 JDBC 连接将表提取到 Spark 环境中。 若要详细了解并行读取，请参阅[管理平行度](#manage-parallelism)。

```scala
val employees_table = spark.read.jdbc(jdbcUrl, "employees", connectionProperties)
```

Spark 会自动从数据库表中读取架构，并将其类型映射回 Spark SQL 类型。

```scala
employees_table.printSchema
```

可针对此 JDBC 表运行查询：

```scala
display(employees_table.select("age", "salary").groupBy("age").avg("salary"))
```

## <a name="write-data-to-jdbc"></a>将数据写入 JDBC

本部分说明如何通过名为 `diamonds` 的现有 Spark SQL 表将数据写入数据库。

```sql
select * from diamonds limit 5
```

下面的代码将数据保存到名为 `diamonds` 的数据库表中。 若将列命名为保留关键字，可能会触发异常。 示例表包含名为 `table` 的列，因此可在将其推送到 JDBC API 之前，先将其重命名为 `withColumnRenamed()`。

```scala
spark.table("diamonds").withColumnRenamed("table", "table_number")
     .write
     .jdbc(jdbcUrl, "diamonds", connectionProperties)
```

Spark 会自动创建一个数据库表，其中包含根据数据帧架构确定的相应架构。

默认行为是创建一个新表，并在已存在同名的表时引发错误消息。 可使用 Spark SQL `SaveMode` 功能更改此行为。 以下示例介绍了如何在表中追加更多行：

```scala
import org.apache.spark.sql.SaveMode

spark.sql("select * from diamonds limit 10").withColumnRenamed("table", "table_number")
  .write
  .mode(SaveMode.Append) // <--- Append to the existing table
  .jdbc(jdbcUrl, "diamonds", connectionProperties)
```

还可覆盖现有表：

```scala
spark.table("diamonds").withColumnRenamed("table", "table_number")
  .write
  .mode(SaveMode.Overwrite) // <--- Overwrite the existing table
  .jdbc(jdbcUrl, "diamonds", connectionProperties)
```

## <a name="push-down-a-query-to-the-database-engine"></a>将查询向下推送到数据库引擎

可将整个查询向下推送到数据库，且只返回结果。 `table` 参数标识要读取的 JDBC 表。 可使用 SQL 查询 `FROM` 子句中有效的任何内容。

```scala
// Note: The parentheses are required.
val pushdown_query = "(select * from employees where emp_no < 10008) emp_alias"
val df = spark.read.jdbc(url=jdbcUrl, table=pushdown_query, properties=connectionProperties)
display(df)
```

## <a name="push-down-optimization"></a>向下推送优化

除了引入整个表之外，还可将查询向下推送到数据库，从而利用它进行处理且只返回结果。

```scala
// Explain plan with no column selection returns all columns
spark.read.jdbc(jdbcUrl, "diamonds", connectionProperties).explain(true)
```

可通过 `DataFrame` 方法删除列并将查询谓词下推到数据库。

```scala
// Explain plan with column selection will prune columns and just return the ones specified
// Notice that only the 3 specified columns are in the explain plan
spark.read.jdbc(jdbcUrl, "diamonds", connectionProperties).select("carat", "cut", "price").explain(true)

// You can push query predicates down too
// Notice the filter at the top of the physical plan
spark.read.jdbc(jdbcUrl, "diamonds", connectionProperties).select("carat", "cut", "price").where("cut = 'Good'").explain(true)
```

## <a name="manage-parallelism"></a><a id="jdbc-parallelism"> </a><a id="manage-parallelism"> </a>管理并行度

在 Spark UI 中，`numPartitions` 指示的是已启动的任务数。 每个任务分布在各执行器中，这可能会增加通过 JDBC 接口进行读取和写入的并行度。 若要了解其他有助于提高性能的参数（例如 `fetchsize`），请参阅 [Spark SQL 编程指南](https://spark.apache.org/docs/latest/sql-programming-guide.html#jdbc-to-other-databases)。

### <a name="jdbc-reads"></a>JDBC 读取

可基于数据集的列值来提供拆分边界。

这些选项可指定读取的并行度。 如果指定了这些选项中的任何一个，则必须指定全部选项。 `lowerBound` 和 `upperBound` 决定分区步幅，不会筛选表中的行。 因此，Spark 分区会返回表中的所有行。

下面的示例使用 `columnName`、`lowerBound`、`upperBound` 和 `numPartitions` 参数在 `emp_no` 列上的执行器之间拆分表读取。

```scala
val df = (spark.read.jdbc(url=jdbcUrl,
  table="employees",
  columnName="emp_no",
  lowerBound=1L,
  upperBound=100000L,
  numPartitions=100,
  connectionProperties=connectionProperties))
display(df)
```

### <a name="jdbc-writes"></a>JDBC 写入

Spark 的分区指示用于通过 JDBC API 推送数据的连接数。 可通过调用 `coalesce(<N>)` 或 `repartition(<N>)` 来控制并行度，具体取决于现有的分区数。 减少分区数时调用 `coalesce`，增加分区数时调用 `repartition`。

```scala
import org.apache.spark.sql.SaveMode

val df = spark.table("diamonds")
println(df.rdd.partitions.length)

// Given the number of partitions above, you can reduce the partition value by calling coalesce() or increase it by calling repartition() to manage the number of connections.
df.repartition(10).write.mode(SaveMode.Append).jdbc(jdbcUrl, "diamonds", connectionProperties)
```

## <a name="python-example"></a>Python 示例

下面的 Python 示例介绍了一些与 Scala 执行的相同的任务。

### <a name="create-the-jdbc-url"></a><a id="create-the-jdbc-url"> </a><a id="python-jdbc-example"> </a>创建 JDBC URL

```python
jdbcHostname = "<hostname>"
jdbcDatabase = "employees"
jdbcPort = 1433
jdbcUrl = "jdbc:sqlserver://{0}:{1};database={2};user={3};password={4}".format(jdbcHostname, jdbcPort, jdbcDatabase, username, password)
```

与前面的 Scala 示例类似，可传入包含凭据和驱动程序类的字典。

```python
jdbcUrl = "jdbc:sqlserver://{0}:{1};database={2}".format(jdbcHostname, jdbcPort, jdbcDatabase)
connectionProperties = {
  "user" : jdbcUsername,
  "password" : jdbcPassword,
  "driver" : "com.microsoft.sqlserver.jdbc.SQLServerDriver"
}
```

### <a name="push-down-a-query-to-the-database-engine"></a>将查询向下推送到数据库引擎

```python
pushdown_query = "(select * from employees where emp_no < 10008) emp_alias"
df = spark.read.jdbc(url=jdbcUrl, table=pushdown_query, properties=connectionProperties)
display(df)
```

### <a name="read-from-jdbc-connections-across-multiple-workers"></a>从 JDBC 连接跨多个工作器节点进行读取

```python
df = spark.read.jdbc(url=jdbcUrl, table="employees", column="emp_no", lowerBound=1, upperBound=100000, numPartitions=100)
display(df)
```

## <a name="spark-sql-example"></a>Spark SQL 示例

可定义一个使用 JDBC 连接的 Spark SQL 表或视图。 有关详细信息，请参阅[创建表](../../spark/latest/spark-sql/language-manual/create-table.md)和[创建视图](../../spark/latest/spark-sql/language-manual/create-view.md)。

```sql
CREATE TABLE <jdbcTable>
USING org.apache.spark.sql.jdbc
OPTIONS (
  url "jdbc:<databaseServerType>://<jdbcHostname>:<jdbcPort>",
  dbtable "<jdbcDatabase>.atable",
  user "<jdbcUsername>",
  password "<jdbcPassword>"
)
```

使用 Spark SQL 将数据追加到数据库表中：

```sql
INSERT INTO diamonds
SELECT * FROM diamonds LIMIT 10 -- append 10 records to the table

SELECT count(*) record_count FROM diamonds --count increased by 10
```

使用 Spark SQL 覆盖数据库中的数据。 这会让数据库删除并创建 `diamonds` 表：

```sql
INSERT OVERWRITE TABLE diamonds
SELECT carat, cut, color, clarity, depth, TABLE AS table_number, price, x, y, z FROM diamonds

SELECT count(*) record_count FROM diamonds --count returned to original value (10 less)
```

## <a name="optimize-performance-when-reading-data"></a>优化读取数据时的性能

如果尝试从外部 JDBC 数据库中读取数据，但速度较慢，可参考本部分中介绍的一些性能改进建议。

### <a name="determine-whether-the-jdbc-unload-is-occurring-in-parallel"></a>确定 JDBC 卸载是否并行进行

若要并行加载数据，Spark JDBC 数据源必须配置有适当的分区信息，以便能够向外部数据库发起多个并行查询。 如果没有配置分区，则将在驱动程序上使用单个 JDBC 查询提取所有数据，这会导致驱动程序引发 OOM 异常。

下面是未配置分区的 JDBC 读取示例：

> [!div class="mx-imgBorder"]
> ![无分区的 JDBC 读取](../../_static/images/data-sources/jdbc-read-no-partition.png)

有两个 API 用于指定分区：高级别和低级别。

[高级别 API](#jdbc-parallelism) 使用数值列的名称 (`columnName`)、两个范围终结点（`lowerBound` 和 `upperBound`）以及目标 `numPartitions`，并通过将指定范围平均拆分给 `numPartitions` 个任务来生成 Spark 任务。 如果数据库表具有已编制索引的数值列且值均匀分布，则非常使用此类 API；如果数值列的值极不均匀，则不建议使用它，因为这会导致拆分出的任务不均衡。

[低级别 API](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameReader@jdbc(url:String,table:String,columnName:String,lowerBound:Long,upperBound:Long,numPartitions:Int,connectionProperties:java.util.Properties):org.apache.spark.sql.DataFrame) 可在 Scala 中访问，它接受可用于定义自定义分区的 `WHERE` 条件的数组：这适用于对非数值列进行分区或用于处理分布倾斜的情况。 定义自定义分区时，如果分区列可为空，请记得考虑 `NULL`。 建议不要使用超过两列来手动定义分区，因为编写边界谓词需要的逻辑要复杂得多。

下面是已配置分区的 JDBC 读取示例。 请注意添加了一个数值列（`partitionColumn`：这是将 `columnName` 作为 JDBC 源选项进行传递的方式）、两个范围终结点（`lowerBound` 和 `upperBound`），以及用于指定最大分区数的 `numPartitions` 参数。

> [!div class="mx-imgBorder"]
> ![带有分区的 JDBC 读取](../../_static/images/data-sources/jdbc-read-partitioned.png)

有关详细信息，请参阅[管理并行度](#jdbc-parallelism)。

### <a name="tune-the-jdbc-fetchsize-parameter"></a>优化 JDBC `fetchSize` 参数

JDBC 驱动程序有一个 `fetchSize` 参数，它控制一次从远程 JDBC 数据库中提取的行数。 如果此值设置得太低，那么为了获取完整的结果集，工作负载可能会因为 Spark 和外部数据库之间往返的请求数过多而出现延迟。 如果此值过高，则可能会引发 OOM。 最佳值将依赖于工作负载（因为它依赖于结果架构、结果中的字符串大小等因素），但在默认值的基础上稍微提高一些就可能获得巨大的性能提升。

Oracle 的 `fetchSize` 默认为 10。 稍微增加一点，提高到 100，就能实现大幅性能提升，如果继续增加到更高的值（如 2000），还可以提高性能。 例如： 。

```java
PreparedStatement stmt = null;
ResultSet rs = null;

try {
  stmt = conn. prepareStatement("select a, b, c from table");
  stmt.setFetchSize(100);

  rs = stmt.executeQuery();
  while (rs.next()) {
    ...
  }
}
```

请参阅[加快 Java 运行速度](https://makejavafaster.blogspot.com/2015/06/jdbc-fetch-size-performance.html)，更深入地了解 Oracle JDBC 驱动程序的这一优化参数。

### <a name="consider-the-impact-of-indexes"></a>考虑索引的影响

如果要进行并行读取（使用分区技术之一），Spark 会对 JDBC 数据库发出并发查询。 如果这些查询最终需要进行全表扫描，则可能会在远程数据库中遭遇瓶颈，且会极大地减缓运行速度。 因此，在选择分区依据列时应考虑索引的影响，以便在选择后能合理有效地并行执行各个分区的查询。

> [!IMPORTANT]
>
> 请确保数据库在分区依据列上有索引。

如果未在源表上定义单列索引，那么仍然可以选择组合索引中的前导（最左）列作为分区依据列。 当只有组合索引可用时，大多数数据库在使用前导（最左）列进行搜索时可以使用串联索引。 因此，多列索引中的前导列也可用作分区依据列。

### <a name="consider-whether-the-number-of-partitions-is-appropriate"></a>考虑分区数是否合适

在从外部数据库中读取数据时，如果分区数过多，可能会由于查询过多而导致数据库重载。 大多数 DBMS 系统对并发连接数都是有限制的。 首先，将分区数设置为接近 Spark 集群中核心/任务槽数量的值，以便尽可能地提高并行度，同时将查询总数限制在合理的限度内。 如果在提取 JDBC 行后对并行度有极大的需求（因为在 Spark 中执行一些占用 CPU 的操作），但又不想向数据库发出过多的并发查询，那么请考虑使用更低的 `numPartitions` 来读取 JDBC，然后在 Spark 中执行显式 `repartition()`。

### <a name="consider-database-specific-tuning-techniques"></a>考虑特定于数据库的优化技术

数据库供应商可能提供了关于 ETL 和批量访问工作负载的性能优化指南。