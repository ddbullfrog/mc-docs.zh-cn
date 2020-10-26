---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/30/2020
title: 表批量读取和写入 - Azure Databricks
description: 了解如何对 Delta 表执行批量读取和写入操作。
ms.openlocfilehash: 90f6d3e505d1f3891b0631def09b37a438ac1025
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121893"
---
# <a name="table-batch-reads-and-writes"></a>表批量读取和写入

Delta Lake 支持 Apache Spark 数据帧读取和写入 API 提供的大部分选项，这些选项可用于对表执行批量读取和写入操作。

有关 Delta Lake SQL 命令的信息，请参阅[适用于 SQL 开发人员的 Azure Databricks](../spark/latest/spark-sql/index.md)。

## <a name="create-a-table"></a><a id="create-a-table"> </a><a id="ddlcreatetable"> </a>创建表

Delta Lake 支持使用 `DataFrameWriter`（[Scala 或 Java](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameWriter)/[Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrameWriter)）直接基于路径创建表。 Delta Lake 还支持使用标准 DDL `CREATE TABLE` 在元存储中创建表。 使用 Delta Lake 在元存储中创建表时，它会将表数据的位置存储在元存储中。 通过此指针，其他用户可以更轻松地发现和引用数据，无需担心数据的确切存储位置。 不过，元存储不是表中有效内容的事实来源。 那仍然是 Delta Lake 的职责。

### <a name="sql"></a>SQL

```sql
-- Create table in the metastore
CREATE TABLE events (
  date DATE,
  eventId STRING,
  eventType STRING,
  data STRING)
USING DELTA
```

### <a name="python"></a>Python

```python
df.write.format("delta").saveAsTable("events")      # create table in the metastore

df.write.format("delta").save("/mnt/delta/events")  # create table by path
```

### <a name="scala"></a>Scala

```scala
df.write.format("delta").saveAsTable("events")      // create table in the metastore

df.write.format("delta").save("/mnt/delta/events")  // create table by path
```

在 Databricks Runtime 7.0 及更高版本中，可以使用 DataFrameWriterV2 接口创建 Delta 表。 SQL 还支持在路径上创建表，无需在 Hive 元存储中创建条目。

### <a name="sql"></a>SQL

```sql
-- Create a table by path
CREATE OR REPLACE TABLE delta.`/mnt/delta/events` (
  date DATE,
  eventId STRING,
  eventType STRING,
  data STRING)
USING DELTA
PARTITIONED BY (date);

-- Create a table in the metastore
CREATE OR REPLACE TABLE events (
  date DATE,
  eventId STRING,
  eventType STRING,
  data STRING)
USING DELTA
PARTITIONED BY (date);
```

### <a name="scala"></a>Scala

```scala
df.writeTo("delta.`/mnt/delta/events`").using("delta").partitionedBy("date").createOrReplace() // create table by path

df.writeTo("events").using("delta").partitionedBy("date").createOrReplace()                   // create table in the metastore
```

### <a name="partition-data"></a>将数据分区

你可以对数据进行分区，以加速其谓词涉及分区列的查询或 DML。
若要在创建 Delta 表时对数据进行分区，请指定按列分区。 常见的模式是按日期进行分区，例如：

#### <a name="sql"></a>SQL

```sql
-- Create table in the metastore
CREATE TABLE events (
 date DATE,
 eventId STRING,
 eventType STRING,
 data STRING)
USING DELTA
PARTITIONED BY (date)
LOCATION '/mnt/delta/events'
```

#### <a name="python"></a>Python

```python
df.write.format("delta").partitionBy("date").saveAsTable("events")      # create table in the metastore

df.write.format("delta").partitionBy("date").save("/mnt/delta/events")  # create table by path
```

#### <a name="scala"></a>Scala

```scala
df.write.format("delta").partitionBy("date").saveAsTable("events")      // create table in the metastore

df.write.format("delta").partitionBy("date").save("/mnt/delta/events")  // create table by path
```

### <a name="control-data-location"></a>控制数据位置

若要控制 Delta 表文件的位置，可以选择将 `LOCATION` 指定为 DBFS 上的路径。

使用指定的 `LOCATION` 创建的表被视为不受元存储管理。  与不指定路径的托管表不同，非托管表的文件在你 `DROP` 表时不会被删除。

如果运行 `CREATE TABLE` 时指定的 `LOCATION` 已包含使用 Delta Lake 存储的数据，则 Delta Lake 会执行以下操作：

* 如果仅指定了表名称和位置，例如：

  ```sql
  CREATE TABLE events
  USING DELTA
  LOCATION '/mnt/delta/events'
  ```

  Hive 元存储中的表会自动继承现有数据的架构、分区和表属性。 此功能可用于将数据“导入”到元存储中。

* 如果你指定了任何配置（架构、分区或表属性），则 Delta Lake 会验证指定的内容是否与现有数据的配置完全匹配。

  > [!IMPORTANT]
  >
  > 如果指定的配置与数据的配置并非完全匹配，则 Delta Lake 会引发一个描述差异的异常。

## <a name="read-a-table"></a><a id="deltadataframereads"> </a><a id="read-a-table"> </a>读取表

可以通过指定一个路径将 Delta 表作为数据帧加载：

### <a name="sql"></a>SQL

```sql
SELECT * FROM events   -- query table in the metastore

SELECT * FROM delta.`/mnt/delta/events`  -- query table by path
```

### <a name="python"></a>Python

```python
spark.table("events")    # query table in the metastore

spark.read.format("delta").load("/mnt/delta/events")  # query table by path
```

### <a name="scala"></a>Scala

```scala
spark.table("events")      // query table in the metastore

spark.read.format("delta").load("/mnt/delta/events")  // create table by path
```

返回的数据帧会自动读取表的最新快照来进行任何查询；你永远不需要运行 `REFRESH TABLE`。 如果查询中存在适用的谓词，则 Delta Lake 会自动使用分区和统计信息来读取最少量的数据。

### <a name="query-an-older-snapshot-of-a-table-time-travel"></a><a id="deltatimetravel"> </a><a id="query-an-older-snapshot-of-a-table-time-travel"> </a>查询表的旧快照（按时间顺序查看）

#### <a name="in-this-section"></a>本节内容：

* [语法](#syntax)
* [数据保留](#data-retention)
* [示例](#examples)

Delta Lake 按时间顺序查看允许你查询 Delta 表的旧快照。 按时间顺序查看有许多用例，包括：

* 重新创建分析、报表或输出（例如，机器学习模型的输出）。 这对于调试或审核非常有用，尤其是在管控行业中。
* 编写复杂的时态查询。
* 修复数据中的错误。
* 为针对快速变化表的一组查询提供快照隔离。

本部分介绍了支持用于查询旧版表的方法、数据保留考虑事项，并提供了示例。

#### <a name="syntax"></a>语法

本部分介绍了如何查询 Delta 表的较旧版本。

##### <a name="in-this-section"></a>本节内容：

* [SQL `AS OF` 语法](#sql-as-of-syntax)
* [DataFrameReader 选项](#dataframereader-options)
* [`@` 语法](#-syntax)

##### <a name="sql-as-of-syntax"></a>SQL `AS OF` 语法

```sql
SELECT * FROM events TIMESTAMP AS OF timestamp_expression
SELECT * FROM events VERSION AS OF version
```

* `timestamp_expression` 可以是下列项中的任意一项：
  * `'2018-10-18T22:15:12.013Z'`，即可以强制转换为时间戳的字符串
  * `cast('2018-10-18 13:36:32 CEST' as timestamp)`
  * `'2018-10-18'`，即日期字符串
  * 在 Databricks Runtime 6.6 及更高版本中：
    * `current_timestamp() - interval 12 hours`
    * `date_sub(current_date(), 1)`
    * 本身就是时间戳或可强制转换为时间戳的任何其他表达式
* `version` 是可以从 `DESCRIBE HISTORY events` 的输出中获取的 long 值。

`timestamp_expression` 和 `version` 都不能是子查询。

##### <a name="dataframereader-options"></a>DataFrameReader 选项

使用 DataFrameReader 选项，可以从固定到表的特定版本的 Delta 表创建数据帧。

```python
df1 = spark.read.format("delta").option("timestampAsOf", timestamp_string).load("/mnt/delta/events")
df2 = spark.read.format("delta").option("versionAsOf", version).load("/mnt/delta/events")
```

对于 `timestamp_string`，只接受日期或时间戳字符串。 例如，`"2019-01-01"` 和 `"2019-01-01T00:00:00.000Z"`。

常见的模式是在执行 Azure Databricks 作业的整个过程中使用 Delta 表的最新状态来更新下游应用程序。

由于 Delta 表会自动更新，因此，如果基础数据进行了更新，则在进行多次调用时，从 Delta 表加载的数据帧可能会返回不同的结果。 通过使用按时间顺序查看，你可以修复多次调用时数据帧返回的数据：

```python
latest_version = spark.sql("SELECT max(version) FROM (DESCRIBE HISTORY delta.`/mnt/delta/events`)").collect()
df = spark.read.format("delta").option("versionAsOf", latest_version[0][0]).load("/mnt/delta/events")
```

##### <a name="-syntax"></a>`@` 语法

你可以采用一个参数化管道，其中管道的输入路径是作业的参数。 在作业执行完以后，你可能需要在将来的某个时间重新生成输出。 在这种情况下，可以使用 `@` 语法来指定时间戳或版本。 时间戳必须采用 `yyyyMMddHHmmssSSS` 格式。 你可以通过在版本前附加一个 `v` 在 `@` 后指定版本。 例如，若要查询表 `events` 的版本 `123`，请指定 `events@v123`。

###### <a name="sql"></a>SQL

```sql
SELECT * FROM events@20190101000000000
SELECT * FROM events@v123
```

###### <a name="python"></a>Python

```python
spark.read.format("delta").load("/mnt/delta/events@20190101000000000") # table on 2019-01-01 00:00:00.000
spark.read.format("delta").load("/mnt/delta/events@v123")              # table on version 123
```

###### <a name="python"></a>Python

```python
spark.read.format("delta").load("/mnt/delta/events@20190101000000000") // table on 2019-01-01 00:00:00.000
spark.read.format("delta").load("/mnt/delta/events@v123")              // table on version 123
```

#### <a name="data-retention"></a>数据保留

默认情况下，Delta 表将提交历史记录保留 30 天。 这意味着，你可以指定 30 天前的版本。 但是，有以下注意事项：

* 你未对 Delta 表运行 [VACUUM](delta-utility.md#vacuum)。 如果你运行了 `VACUUM`，则无法恢复到早于默认的 7 天数据保留期的版本。

可以使用以下[表属性](#table-properties)配置保留期：

* `delta.logRetentionDuration = "interval <interval>"`：控制表的历史记录的保留时间长度。 每次写入检查点时，Azure Databricks 会自动清除早于保留间隔的日志条目。 如果将此配置设置为足够大的值，则会保留许多日志条目。 这应当不会影响性能，因为针对日志的操作时间恒定。 针对历史记录的操作是并行的（但会随着日志大小的增加而变得更为昂贵）。 默认值为 `interval 30 days`。
* `delta.deletedFileRetentionDuration = "interval <interval>"`：对文件必须已删除多长时间才能成为 `VACUUM` 的候选对象进行控制。 默认值为 `interval 7 days`。 若要访问 30 天的历史数据，请设置 `delta.deletedFileRetentionDuration = "interval 30 days"`。 此设置可能会导致你的存储成本增加。

  > [!NOTE]
  >
  > `VACUUM` 不清除日志文件；在写入检查点后，会自动清除日志文件。

若要按时间顺序查看以前的某个版本，必须同时保留该版本的日志文件和数据文件。

#### <a name="examples"></a>示例

* 为用户 `111` 修复对表的意外删除问题：

  ```sql
  INSERT INTO my_table
    SELECT * FROM my_table TIMESTAMP AS OF date_sub(current_date(), 1)
    WHERE userId = 111
  ```

* 修复对表的意外错误更新：

  ```sql
  MERGE INTO my_table target
    USING my_table TIMESTAMP AS OF date_sub(current_date(), 1) source
    ON source.userId = target.userId
    WHEN MATCHED THEN UPDATE SET *
  ```

* 查询在过去一周内增加的新客户的数量。

  ```sql
  SELECT count(distinct userId) - (
    SELECT count(distinct userId)
    FROM my_table TIMESTAMP AS OF date_sub(current_date(), 7))
  ```

## <a name="write-to-a-table"></a><a id="deltadataframewrites"> </a><a id="write-to-a-table"> </a>写入到表

### <a name="append"></a><a id="append"> </a><a id="batch-append"> </a>追加

使用 `append` 模式，可以将新数据以原子方式添加到现有 Delta 表中：

#### <a name="sql"></a>SQL

```sql
INSERT INTO events SELECT * FROM newEvents
```

#### <a name="python"></a>Python

```python
df.write.format("delta").mode("append").save("/mnt/delta/events")
df.write.format("delta").mode("append").saveAsTable("events")
```

#### <a name="scala"></a>Scala

```scala
df.write.format("delta").mode("append").save("/mnt/delta/events")
df.write.format("delta").mode("append").saveAsTable("events")
```

### <a name="overwrite"></a>Overwrite

若要以原子方式替换表中的所有数据，可以使用 `overwrite` 模式：

#### <a name="sql"></a>SQL

```sql
INSERT OVERWRITE TABLE events SELECT * FROM newEvents
```

#### <a name="python"></a>Python

```python
df.write.format("delta").mode("overwrite").save("/mnt/delta/events")
df.write.format("delta").mode("overwrite").saveAsTable("events")
```

#### <a name="scala"></a>Scala

```scala
df.write.format("delta").mode("overwrite").save("/mnt/delta/events")
df.write.format("delta").mode("overwrite").saveAsTable("events")
```

使用数据帧，你还可以有选择性地只覆盖与分区列上的谓词匹配的数据。 以下命令以原子方式将一月份替换为 `df` 中的数据：

#### <a name="python"></a>Python

```python
df.write \
  .format("delta") \
  .mode("overwrite") \
  .option("replaceWhere", "date >= '2017-01-01' AND date <= '2017-01-31'") \
  .save("/mnt/delta/events")
```

#### <a name="scala"></a>Scala

```scala
df.write
  .format("delta")
  .mode("overwrite")
  .option("replaceWhere", "date >= '2017-01-01' AND date <= '2017-01-31'")
  .save("/mnt/delta/events")
```

此示例代码将 `df` 中的数据写出，验证它是否位于指定的分区中，并执行原子替换。

> [!NOTE]
>
> 与 Apache Spark 中的文件 API 不同，Delta Lake 会记住并强制实施表的架构。 这意味着，默认情况下，覆盖不会替换现有表的架构。

有关 Delta Lake 在更新表方面的支持，请参阅[表删除、更新和合并](delta-update.md)。

### <a name="set-user-defined-commit-metadata"></a><a id="set-user-defined-commit-metadata"> </a><a id="user-metadata"> </a>设置用户定义的提交元数据

可以使用 DataFrameWriter 选项 `userMetadata` 或 SparkSession 配置 `spark.databricks.delta.commitInfo.userMetadata`，将用户定义的字符串指定为这些操作所进行的提交中的元数据。 如果同时指定了两个参数，则此选项将优先。 此用户定义的元数据在[历史记录](delta-utility.md#describe-history)操作中是可读的。

#### <a name="sql"></a>SQL

```sql

SET spark.databricks.delta.commitInfo.userMetadata=overwritten-for-fixing-incorrect-data
INSERT OVERWRITE events SELECT * FROM newEvents
```

#### <a name="python"></a>Python

```python
df.write.format("delta") \
  .mode("overwrite") \
  .option("userMetadata", "overwritten-for-fixing-incorrect-data") \
  .save("/mnt/delta/events")
```

#### <a name="scala"></a>Scala

```scala
df.write.format("delta")
  .mode("overwrite")
  .option("userMetadata", "overwritten-for-fixing-incorrect-data")
  .save("/mnt/delta/events")
```

## <a name="schema-validation"></a>架构验证

Delta Lake 会自动验证正在写入的数据帧的架构是否与表的架构兼容。 Delta Lake 使用以下规则来确定从数据帧到表的写入是否兼容：

* 所有数据帧列都必须存在于目标表中。 如果数据帧中有表中不存在的列，则会引发异常。 表中存在但数据帧中不存在的列将设置为 NULL。
* 数据帧列数据类型必须与目标表中的列数据类型匹配。 如果它们不匹配，则会引发异常。
* 数据帧列名称不能仅通过大小写来区分。 这意味着不能在同一个表中定义诸如“Foo”和“foo”之类的列。 尽管可以在区分大小写或不区分大小写（默认）模式下使用 Spark，但 Parquet 在存储和返回列信息时区分大小写。 在存储架构时，Delta Lake 保留但不区分大小写，并采用此限制来避免潜在的错误、数据损坏或丢失问题。

Delta Lake 支持使用 DDL 显式添加新列并自动更新架构。

如果你指定其他选项（例如 `partitionBy`）与追加模式结合使用，则 Delta Lake 会验证它们是否匹配，在不匹配时会引发错误。  未提供 `partitionBy` 时，会在对现有数据分区之后自动进行追加。

> [!NOTE]
>
> 在 Databricks Runtime 7.0 及更高版本中，`INSERT` 语法提供了架构强制实施，并支持架构演变。 如果列的数据类型不能安全地强制转换为 Delta Lake 表的数据类型，则会引发运行时异常。 如果启用了[架构演变](#automatic-schema-update)，则新列可以作为架构的最后一列（或嵌套列）存在，以便架构得以演变。

## <a name="update-table-schema"></a><a id="ddlschema"> </a><a id="update-table-schema"> </a>更新表架构

Delta Lake 允许你更新表的架构。 支持下列类型的更改：

* 添加新列（在任意位置）
* 重新排列现有列

你可以使用 DDL 显式地或使用 DML 隐式地进行这些更改。

> [!IMPORTANT]
>
> 更新 Delta 表架构时，从该表进行读取的流会终止。 如果你希望流继续进行，必须重启它。

有关建议的方法，请参阅[生产中的结构化流式处理](../spark/latest/structured-streaming/production.md)。

### <a name="explicitly-update-schema"></a><a id="explicit-schema-update"> </a><a id="explicitly-update-schema"> </a>显式更新架构

你可以使用以下 DDL 显式更改表的架构。

#### <a name="add-columns"></a>添加列

```sql
ALTER TABLE table_name ADD COLUMNS (col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name], ...)
```

默认情况下，为 Null 性为 `true`。

若要将列添加到嵌套字段，请使用：

```sql
ALTER TABLE table_name ADD COLUMNS (col_name.nested_col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name], ...)
```

##### <a name="example"></a>示例

如果运行 `ALTER TABLE boxes ADD COLUMNS (colB.nested STRING AFTER field1)` 之前的架构为：

```
- root
| - colA
| - colB
| +-field1
| +-field2
```

则运行之后的架构为：

```
- root
| - colA
| - colB
| +-field1
| +-nested
| +-field2
```

> [!NOTE]
>
> 仅支持为结构添加嵌套列。 不支持数组和映射。

#### <a name="change-column-comment-or-ordering"></a>更改列注释或排序

```sql
ALTER TABLE table_name CHANGE [COLUMN] col_name col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name]
```

若要更改嵌套字段中的列，请使用：

```sql
ALTER TABLE table_name CHANGE [COLUMN] col_name.nested_col_name nested_col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name]
```

##### <a name="example"></a>示例

如果运行 `ALTER TABLE boxes CHANGE COLUMN colB.field2 field2 STRING FIRST` 之前的架构为：

```
- root
| - colA
| - colB
| +-field1
| +-field2
```

则运行之后的架构为：

```
- root
| - colA
| - colB
| +-field2
| +-field1
```

#### <a name="replace-columns"></a>替换列

```sql
ALTER TABLE table_name REPLACE COLUMNS (col_name1 col_type1 [COMMENT col_comment1], ...)
```

##### <a name="example"></a>示例

运行以下 DSL 时：

```sql
ALTER TABLE boxes REPLACE COLUMNS (colC STRING, colB STRUCT<field2:STRING, nested:STRING, field1:STRING>, colA STRING)
```

如果运行之前的架构为：

```
- root
| - colA
| - colB
| +-field1
| +-field2
```

则运行之后的架构为：

```
- root
| - colC
| - colB
| +-field2
| +-nested
| +-field1
| - colA
```

#### <a name="change-column-type-or-name"></a><a id="change-column-type"> </a><a id="change-column-type-or-name"> </a>更改列类型或名称

更改列的类型或名称或者删除列需要重写该表。 为此，请使用 `overwriteSchema` 选项：

##### <a name="change-a-column-type"></a>更改列类型

```python
spark.read.table(...)
  .withColumn("date", col("date").cast("date"))
  .write
  .format("delta")
  .mode("overwrite")
  .option("overwriteSchema", "true")
  .saveAsTable(...)
```

##### <a name="change-a-column-name"></a>更改列名称

```python
spark.read.table(...)
  .withColumnRenamed("date", "date_created")
  .write
  .format("delta")
  .mode("overwrite")
  .option("overwriteSchema", "true")
  .saveAsTable(...)
```

### <a name="automatic-schema-update"></a>自动架构更新

Delta Lake 可以在 DML 事务（追加或覆盖）中自动更新表的架构，并使架构与要写入的数据兼容。

#### <a name="add-columns"></a>添加列

在以下情况下，会自动将数据帧中存在但表中缺少的列添加为写入事务的一部分：

* `write` 或 `writeStream` 具有 `.option("mergeSchema", "true")`
* `spark.databricks.delta.schema.autoMerge.enabled` 为 `true`

如果同时指定了这两个选项，则优先使用 `DataFrameWriter` 中的选项。 添加的列将追加到它们所在结构的末尾。 追加新列时，会保留大小写。

> [!NOTE]
>
> * 当启用了[表访问控制](../security/access-control/table-acls/object-privileges.md)时，`mergeSchema` 不受支持（因为它会将需要 `MODIFY` 的请求提升为需要 `ALL PRIVILEGES` 的请求）。
> * `mergeSchema` 不能与 `INSERT INTO` 或 `.write.insertInto()` 一起使用。

#### <a name="nulltype-columns"></a>`NullType` 列

由于 Parquet 不支持 `NullType`，因此在写入到 Delta 表时会从数据帧中删除 `NullType` 列，但仍会将其存储在架构中。 如果为该列接收到不同的数据类型，则 Delta Lake 会将该架构合并到新的数据类型。 如果 Delta Lake 接收到现有列的 `NullType`，则在写入过程中会保留旧架构并删除新列。

不支持流式处理中的 `NullType`。 由于在使用流式处理时必须设置架构，因此这应该非常罕见。 对于复杂类型（例如 `ArrayType` 和 `MapType`），也不会接受 `NullType`。

## <a name="replace-table-schema"></a>替换表架构

默认情况下，覆盖表中的数据不会覆盖架构。 在不使用 `replaceWhere` 的情况下使用 `mode("overwrite")` 来覆盖表时，你可能还希望覆盖写入的数据的架构。 你可以通过将 `overwriteSchema` 选项设置为 `true` 来替换表的架构和分区：

```python
df.write.option("overwriteSchema", "true")
```

## <a name="views-on-tables"></a>表中的视图

Delta Lake 支持基于 Delta 表创建视图，就像使用数据源表一样。

这些视图集成了[表访问控制](../security/access-control/table-acls/object-privileges.md)，可以实现列级和行级安全性。

处理视图时的主要难题是解析架构。 如果你更改 Delta 表架构，则必须重新创建派生视图，以容纳向该架构添加的任何内容。 例如，如果向 Delta 表中添加一个新列，则必须确保此列在基于该基表构建的相应视图中可用。

## <a name="table-properties"></a>表属性

你可以使用 `CREATE` 和 `ALTER` 中的 `TBLPROPERTIES` 将自己的元数据存储为表属性。

`TBLPROPERTIES` 存储为 Delta 表元数据的一部分。 如果在给定位置已存在 Delta 表，则无法在 `CREATE` 语句中定义新的 `TBLPROPERTIES`。 有关更多详细信息，请参阅[创建表](#ddlcreatetable)。

此外，为了调整行为和性能，Delta Lake 支持某些 Delta 表属性：

* 阻止 Delta 表中的删除和更新：`delta.appendOnly=true`。
* 配置[按时间顺序查看](#deltatimetravel)保留属性：`delta.logRetentionDuration=<interval-string>` 和 `delta.deletedFileRetentionDuration=<interval-string>`。 有关详细信息，请参阅[数据保留](#data-retention)。

* 配置要为其收集统计信息的列的数目：`delta.dataSkippingNumIndexedCols=<number-of-columns>`。 此属性仅对写出的新数据有效。

> [!NOTE]
>
> * 这些是仅有的受支持的带 `delta.` 前缀的表属性。
> * 修改 Delta 表属性是一个写入操作，该操作会与其他[并发写入操作](concurrency-control.md)冲突，导致这些操作失败。 建议仅当不存在对表的并发写入操作时才修改表属性。

你还可以在第一次提交到 Delta 表期间使用 Spark 配置来设置带 `delta.` 前缀的属性。 例如，若要使用属性 `delta.appendOnly=true` 初始化 Delta 表，请将 Spark 配置 `spark.databricks.delta.properties.defaults.appendOnly` 设置为 `true`。 例如： 。

### <a name="sql"></a>SQL

```sql
spark.sql("SET spark.databricks.delta.properties.defaults.appendOnly = true")
```

### <a name="scala"></a>Scala

```scala
spark.conf.set("spark.databricks.delta.properties.defaults.appendOnly", "true")
```

### <a name="python"></a>Python

```python
spark.conf.set("spark.databricks.delta.properties.defaults.appendOnly", "true")
```

## <a name="table-metadata"></a>表元数据

Delta Lake 提供了丰富的用来浏览表元数据的功能。

它支持[显示分区](../spark/latest/spark-sql/language-manual/show-partitions.md)、[显示列](../spark/latest/spark-sql/language-manual/show-columns.md)、[描述表](../spark/latest/spark-sql/language-manual/describe-table.md)，等等。

它还提供了以下独特的命令：

* [`DESCRIBE DETAIL`](#describe-detail)
* [`DESCRIBE HISTORY`](#describe-history)

### `DESCRIBE DETAIL`

提供架构、分区、表大小等方面的信息。 有关详细信息，请参阅[描述详细信息](delta-utility.md#delta-detail)。

### `DESCRIBE HISTORY`

提供出处信息，包括操作、用户等，以及向表的每次写入的操作指标。 表历史记录会保留 30 天。 有关详细信息，请参阅[描述历史记录](delta-utility.md#delta-history)。

[数据边栏](../data/tables.md#view-table-ui)提供此详细的表信息的可视化视图和 Delta 表的历史记录。 除了表架构和示例数据之外，你还可以单击“历史记录”选项卡以查看随 `DESCRIBE HISTORY` 一起显示的表历史记录。

## <a name="notebook"></a>笔记本

有关各种 Delta 表元数据命令的示例，请参阅以下笔记本的末尾：

### <a name="delta-lake-batch-commands-notebook"></a>Delta Lake 批处理命令笔记本

[获取笔记本](../_static/notebooks/delta/quickstart-sql.html)