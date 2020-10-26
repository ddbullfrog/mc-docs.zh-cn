---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/08/2020
title: Delta Lake 快速入门 - Azure Databricks
description: 了解如何完成 Azure Databricks 上的 Delta Lake 的快速入门。
ms.openlocfilehash: 649b2263d6aea50fe6d4688cb5494972cc665267
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121819"
---
# <a name="delta-lake-quickstart"></a>Delta Lake 快速入门

Delta Lake 快速入门概述了使用 Delta Lake 的基础知识。 本快速入门展示了如何生成将 JSON 数据读取到 Delta 表中的管道，以及如何修改表、读取表、显示表历史记录和优化表。

有关演示这些功能的 Azure Databricks 笔记本，请参阅[介绍性笔记本](intro-notebooks.md)。

## <a name="create-a-table"></a>创建表

若要创建 Delta 表，可以使用现有的 Apache Spark SQL 代码，并将格式从 `parquet`、`csv`、`json` 等更改为 `delta`。

对于所有文件类型，都需要将文件读入数据帧并以 `delta` 格式写出：

### <a name="python"></a>Python

```python
events = spark.read.json("/databricks-datasets/structured-streaming/events/")
events.write.format("delta").save("/mnt/delta/events")
spark.sql("CREATE TABLE events USING DELTA LOCATION '/mnt/delta/events/'")
```

### <a name="r"></a>R

```r
library(SparkR)
sparkR.session()

events <- read.json("/databricks-datasets/structured-streaming/events/")
write.df(events, source = "delta", path = "/mnt/delta/events")
sql("CREATE TABLE events USING DELTA LOCATION '/mnt/delta/events/'")
```

### <a name="sql"></a>SQL

```sql
CREATE TABLE events
USING delta
AS SELECT *
FROM json.`/data/events/`
```

这些操作使用从 JSON 数据中推断出的架构来创建新的非托管表。 有关创建新的 Delta 表时可用的完整选项集，请参阅[创建表](delta-batch.md#ddlcreatetable)和[写入到表](delta-batch.md#deltadataframewrites)。

如果源文件采用 Parquet 格式，则可以使用 SQL `Convert to Delta` 语句就地转换文件，以创建非托管表：

```sql
CONVERT TO DELTA parquet.`/mnt/delta/events`
```

### <a name="partition-data"></a>将数据分区

若要加速其谓词涉及分区列的查询，可以对数据进行分区。

#### <a name="python"></a>Python

```python
events = spark.read.json("/databricks-datasets/structured-streaming/events/")
events.write.partitionBy("date").format("delta").save("/mnt/delta/events")
spark.sql("CREATE TABLE events USING DELTA LOCATION '/mnt/delta/events/'")
```

#### <a name="r"></a>R

```r
events <- read.json("/databricks-datasets/structured-streaming/events/")
write.df(events, source = "delta", path = "/mnt/delta/events", partitionBy = "date")
sql("CREATE TABLE events USING DELTA LOCATION '/mnt/delta/events/'")
```

#### <a name="sql"></a>SQL

若要在使用 SQL 创建 Delta 表时对数据进行分区，请指定 `PARTITIONED BY` 列。

```sql
CREATE TABLE events (
  date DATE,
  eventId STRING,
  eventType STRING,
  data STRING)
USING delta
PARTITIONED BY (date)
```

## <a name="modify-a-table"></a>修改表

Delta Lake 支持使用一组丰富的操作来修改表。

### <a name="stream-writes-to-a-table"></a>流式处理到表的写入

你可以使用结构化流式处理将数据写入 Delta 表。  即使有针对表并行运行的其他流或批处理查询，Delta Lake 事务日志也可确保仅处理一次。 默认情况下，流在追加模式下运行，这会将新记录添加到表中。

#### <a name="python"></a>Python

```python
from pyspark.sql.types import *

inputPath = "/databricks-datasets/structured-streaming/events/"

jsonSchema = StructType([ StructField("time", TimestampType(), True), StructField("action", StringType(), True) ])

eventsDF = (
  spark
    .readStream
    .schema(jsonSchema) # Set the schema of the JSON data
    .option("maxFilesPerTrigger", 1) # Treat a sequence of files as a stream by picking one file at a time
    .json(inputPath)
)

(eventsDF.writeStream
  .outputMode("append")
  .option("checkpointLocation", "/mnt/delta/events/_checkpoints/etl-from-json")
  .table("events")
)
```

#### <a name="r"></a>R

```r
inputPath <- "/databricks-datasets/structured-streaming/events/"
tablePath <- "/mnt/delta/events/"

jsonSchema <- structType(structField("time", "timestamp", T), structField("action", "string", T))
eventsStream <- read.stream(
  "json",
  path = inputPath,
  schema = jsonSchema,
  maxFilesPerTrigger = 1)

write.stream(
  eventsStream,
  path = tablePath,
  mode = "append",
  checkpointLocation = "/mnt/delta/events/_checkpoints/etl-from-json")
```

若要详细了解 Delta Lake 与结构化流式处理的集成，请参阅[表流读取和写入](delta-streaming.md)。

### <a name="batch-upserts"></a>批量 upsert

若要将一组更新和插入合并到现有表中，请使用 `MERGE INTO` 语句。 例如，下面的语句将获取一个更新流，并将其合并到 `events` 表中。 如果已存在具有相同 `eventId` 的事件，Delta Lake 会使用给定的表达式更新数据列。 如果没有匹配的事件，Delta Lake 会添加一个新行。

```sql
MERGE INTO events
USING updates
ON events.eventId = updates.eventId
WHEN MATCHED THEN
  UPDATE SET
    events.data = updates.data
WHEN NOT MATCHED
  THEN INSERT (date, eventId, data) VALUES (date, eventId, data)
```

执行 `INSERT` 时必须为表中的每个列指定一个值（例如，当现有数据集中没有匹配行时，必须这样做）。 但是，你不需要更新所有值。

## <a name="read-a-table"></a>读取表

### <a name="in-this-section"></a>本节内容：

* [显示表历史记录](#display-table-history)
* [查询较早版本的表（按时间顺序查看）](#query-an-earlier-version-of-the-table-time-travel)

可以通过指定 DBFS 上的路径 (`"/mnt/delta/events"`) 或表名 (`"events"`) 来访问 Delta 表中的数据：

### <a name="scala"></a>Scala

```scala
val events = spark.read.format("delta").load("/mnt/delta/events")
```

或

```scala
val events = spark.table("events")
```

### <a name="r"></a>R

```r
events <- read.df(path = "/mnt/delta/events", source = "delta")
```

或

```r
events <- tableToDF("events")
```

### <a name="sql"></a>SQL

```sql
SELECT * FROM delta.`/mnt/delta/events`
```

或

```sql
SELECT * FROM events
```

### <a name="display-table-history"></a>显示表历史记录

若要查看表的历史记录，请使用 `DESCRIBE HISTORY` 语句，该语句提供对表进行的每次写入的出处信息，包括表版本、操作、用户等。 请参阅[描述历史记录](delta-utility.md#delta-history)。

### <a name="query-an-earlier-version-of-the-table-time-travel"></a>查询较早版本的表（按时间顺序查看）

Delta Lake 按时间顺序查看允许你查询 Delta 表的旧快照。

对于 `timestamp_string`，只接受日期或时间戳字符串。 例如，`"2019-01-01"` 和 `"2019-01-01'T'00:00:00.000Z"`。

若要查询较早版本的表，请在 `SELECT` 语句中指定版本或时间戳。 例如，若要从上述历史记录中查询版本 0，请使用：

```sql
SELECT * FROM events VERSION AS OF 0
```

或

```sql
SELECT * FROM events TIMESTAMP AS OF '2019-01-29 00:37:58'
```

> [!NOTE]
>
> 由于版本 1 位于时间戳 `'2019-01-29 00:38:10'` 处，因此，若要查询版本 0，可以使用范围 `'2019-01-29 00:37:58'` 到 `'2019-01-29 00:38:09'`（含）中的任何时间戳。

使用 DataFrameReader 选项，可以从固定到表的特定版本的 Delta 表创建数据帧。

```python
df1 = spark.read.format("delta").option("timestampAsOf", timestamp_string).load("/mnt/delta/events")
df2 = spark.read.format("delta").option("versionAsOf", version).load("/mnt/delta/events")
```

有关详细信息，请参阅[查询表的旧快照（按时间顺序查看）](delta-batch.md#deltatimetravel)。

## <a name="optimize-a-table"></a>优化表

对表执行多个更改后，可能会有很多小文件。 为了提高读取查询的速度，你可以使用 `OPTIMIZE` 将小文件折叠为较大的文件：

```sql
OPTIMIZE delta.`/mnt/delta/events`
```

或

```sql
OPTIMIZE events
```

### <a name="z-order-by-columns"></a>按列进行 Z 排序

为了进一步提高读取性能，你可以通过 Z 排序将相关的信息放置在同一组文件中。 Delta Lake 数据跳过算法会自动使用此并置，大幅减少需要读取的数据量。 若要对数据进行 Z 排序，请在 `ZORDER BY` 子句中指定要排序的列。 例如，若要按 `eventType` 并置，请运行：

```sql
OPTIMIZE events
  ZORDER BY (eventType)
```

有关运行 `OPTIMIZE` 时可用的完整选项集，请参阅[压缩（装箱）](optimizations/file-mgmt.md#delta-optimize)。

## <a name="clean-up-snapshots"></a><a id="clean-up"> </a><a id="clean-up-snapshots"> </a>清理快照

Delta Lake 为读取提供快照隔离，这意味着即使其他用户或作业正在查询表，也可以安全地运行 `OPTIMIZE`。 不过，最终你应该清除旧快照。 可以运行 `VACUUM` 命令来执行此操作：

```sql
VACUUM events
```

使用 `RETAIN <N> HOURS` 选项控制最新保留的快照的期限：

```sql
VACUUM events RETAIN 24 HOURS
```

若要详细了解如何有效地使用 `VACUUM`，请参阅[清空](delta-utility.md#delta-vacuum)。