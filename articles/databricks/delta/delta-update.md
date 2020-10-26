---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/05/2020
title: 表删除、更新和合并 - Azure Databricks
description: 了解如何在 Delta 表中删除数据和更新数据。
ms.openlocfilehash: 2b2f56190371858f698c63cf1c54a2980bc769aa
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121876"
---
# <a name="table-deletes-updates-and-merges"></a>表删除、更新和合并

Delta Lake 支持多个语句，以便在 Delta 表中删除数据和更新数据。

## <a name="delete-from-a-table"></a><a id="delete-from-a-table"> </a><a id="delta-delete"> </a>从表中删除

可以从 Delta 表中删除与谓词匹配的数据。 例如，若要删除 `2017` 之前的所有事件，可以运行以下命令：

### <a name="sql"></a>SQL

```sql
DELETE FROM events WHERE date < '2017-01-01'

DELETE FROM delta.`/data/events/` WHERE date < '2017-01-01'
```

### <a name="python"></a>Python

> [!NOTE]
>
> 可在 Databricks Runtime 6.1 及更高版本中使用 Python API。

```python
from delta.tables import *
from pyspark.sql.functions import *

deltaTable = DeltaTable.forPath(spark, "/data/events/")

deltaTable.delete("date < '2017-01-01'")        # predicate using SQL formatted string

deltaTable.delete(col("date") < "2017-01-01")   # predicate using Spark SQL functions
```

### <a name="scala"></a>Scala

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```scala
import io.delta.tables._

val deltaTable = DeltaTable.forPath(spark, "/data/events/")

deltaTable.delete("date < '2017-01-01'")        // predicate using SQL formatted string

import org.apache.spark.sql.functions._
import spark.implicits._

deltaTable.delete(col("date") < "2017-01-01")       // predicate using Spark SQL functions and implicits
```

### <a name="java"></a>Java

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Java API。

```java
import io.delta.tables.*;
import org.apache.spark.sql.functions;

DeltaTable deltaTable = DeltaTable.forPath(spark, "/data/events/");

deltaTable.delete("date < '2017-01-01'");            // predicate using SQL formatted string

deltaTable.delete(functions.col("date").lt(functions.lit("2017-01-01")));   // predicate using Spark SQL functions
```

有关更多详细信息，请参阅 [API 参考](delta-apidoc.md)。

> [!IMPORTANT]
>
> `delete` 可以从最新版本的 Delta 表中删除数据，但是直到显式删除旧的版本后才能从物理存储中删除数据。 有关详细信息，请参阅[清空](delta-utility.md#delta-vacuum)。

> [!TIP]
>
> 如果可能，请在分区的 Delta 表的分区列上提供谓词，因为这样的谓词可以显著加快操作速度。

## <a name="update-a-table"></a><a id="delta-update"> </a><a id="update-a-table"> </a>更新表

可以在 Delta 表中更新与谓词匹配的数据。 例如，若要解决 `eventType` 中的拼写错误，可以运行以下命令：

### <a name="sql"></a>SQL

```sql
UPDATE events SET eventType = 'click' WHERE eventType = 'clck'

UPDATE delta.`/data/events/` SET eventType = 'click' WHERE eventType = 'clck'
```

### <a name="python"></a>Python

> [!NOTE]
>
> 可在 Databricks Runtime 6.1 及更高版本中使用 Python API。

```python
from delta.tables import *
from pyspark.sql.functions import *

deltaTable = DeltaTable.forPath(spark, "/data/events/")

deltaTable.update("eventType = 'clck'", { "eventType": "'click'" } )   # predicate using SQL formatted string

deltaTable.update(col("eventType") == "clck", { "eventType": lit("click") } )   # predicate using Spark SQL functions
```

### <a name="scala"></a>Scala

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```scala
import io.delta.tables._

val deltaTable = DeltaTable.forPath(spark, "/data/events/")

deltaTable.updateExpr(            // predicate and update expressions using SQL formatted string
  "eventType = 'clck'",
  Map("eventType" -> "'click'")

import org.apache.spark.sql.functions._
import spark.implicits._

deltaTable.update(                // predicate using Spark SQL functions and implicits
  col("eventType") === "clck",
  Map("eventType" -> lit("click")));
```

### <a name="java"></a>Java

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```java
import io.delta.tables.*;
import org.apache.spark.sql.functions;
import java.util.HashMap;

DeltaTable deltaTable = DeltaTable.forPath(spark, "/data/events/");

deltaTable.updateExpr(            // predicate and update expressions using SQL formatted string
  "eventType = 'clck'",
  new HashMap<String, String>() {{
    put("eventType", "'click'");
  }}
);

deltaTable.update(                // predicate using Spark SQL functions
  functions.col(eventType).eq("clck"),
  new HashMap<String, Column>() {{
    put("eventType", functions.lit("click"));
  }}
);
```

有关更多详细信息，请参阅 [API 参考](delta-apidoc.md)。

> [!TIP]
>
> 与删除类似，在分区上使用谓词可以显著提高更新操作的速度。

## <a name="upsert-into-a-table-using-merge"></a><a id="delta-merge"> </a><a id="upsert-into-a-table-using-merge"> </a>使用合并操作在表中执行更新插入

可以使用 `merge` 操作将源表、视图或 DataFrame 中的数据更新插入到 Delta 表中。 此操作类似于 SQL `MERGE INTO` 命令，但另外还支持更新、插入和删除操作中的删除操作和附加条件。

假设你有一个 Spark DataFrame，它包含带有 `eventId` 的事件的新数据。 其中一些事件可能已经存在于 `events` 表中。 若要将新数据合并到 `events` 表中，你需要更新匹配的行（即，`eventId` 已经存在）并插入新行（即，`eventId` 不存在）。 可以运行以下查询：

### <a name="sql"></a>SQL

```sql
MERGE INTO events
USING updates
ON events.eventId = updates.eventId
WHEN MATCHED THEN
  UPDATE SET events.data = updates.data
WHEN NOT MATCHED
  THEN INSERT (date, eventId, data) VALUES (date, eventId, data)
```

有关语法的详细信息，请参阅 [MERGE INTO SQL 命令](../spark/latest/spark-sql/language-manual/merge-into.md)。

### <a name="python"></a>Python

```python
from delta.tables import *

deltaTable = DeltaTable.forPath(spark, "/data/events/")

deltaTable.alias("events").merge(
    updatesDF.alias("updates"),
    "events.eventId = updates.eventId") \
  .whenMatchedUpdate(set = { "data" : "updates.data" } ) \
  .whenNotMatchedInsert(values =
    {
      "date": "updates.date",
      "eventId": "updates.eventId",
      "data": "updates.data"
    }
  ) \
  .execute()
```

### <a name="scala"></a>Scala

```scala
import io.delta.tables._
import org.apache.spark.sql.functions._

val updatesDF = ...  // define the updates DataFrame[date, eventId, data]

DeltaTable.forPath(spark, "/data/events/")
  .as("events")
  .merge(
    updatesDF.as("updates"),
    "events.eventId = updates.eventId")
  .whenMatched
  .updateExpr(
    Map("data" -> "updates.data"))
  .whenNotMatched
  .insertExpr(
    Map(
      "date" -> "updates.date",
      "eventId" -> "updates.eventId",
      "data" -> "updates.data"))
  .execute()
```

### <a name="java"></a>Java

```java
import io.delta.tables.*;
import org.apache.spark.sql.functions;
import java.util.HashMap;

Dataset<Row> updatesDF = ...  // define the updates DataFrame[date, eventId, data]

DeltaTable.forPath(spark, "/data/events/")
  .as("events")
  .merge(
    updatesDF.as("updates"),
    "events.eventId = updates.eventId")
  .whenMatched()
  .updateExpr(
    new HashMap<String, String>() {{
      put("data", "events.data");
    }})
  .whenNotMatched()
  .insertExpr(
    new HashMap<String, String>() {{
      put("date", "updates.date");
      put("eventId", "updates.eventId");
      put("data", "updates.data");
    }})
  .execute();
```

有关 Scala、Java 和 Python 语法的详细信息，请查看 [API 参考](delta-apidoc.md)。

### <a name="operation-semantics"></a>操作语义

下面是 `merge` 编程操作的详细说明。

* 可以有任意数量的 `whenMatched` 和 `whenNotMatched` 子句。

  > [!NOTE]
  >
  > 在 Databricks Runtime 7.2 及更低版本中，`merge` 最多可以有 2 个 `whenMatched` 子句和最多 1 个 `whenNotMatched` 子句。

* 当源行根据匹配条件与目标表行匹配时，将执行 `whenMatched` 子句。 这些子句具有以下语义。
  * `whenMatched` 子句最多可以有 1 个 `update` 和 1 个 `delete` 操作。 `merge` 中的 `update` 操作只更新匹配目标行的指定列（类似于 `update` [操作](#delta-update)）。 `delete` 操作删除匹配的行。
  * 每个 `whenMatched` 子句都可以有一个可选条件。 如果存在此子句条件，则仅当该子句条件成立时，才对任何匹配的源-目标行对行执行 `update` 或 `delete` 操作。
  * 如果有多个 `whenMatched` 子句，则将按照指定的顺序对其进行求值（即，子句的顺序很重要）。 除最后一个之外，所有 `whenMatched` 子句都必须具有条件。
  * 如果 2 个 `whenMatched` 子句都具有条件，并且对于匹配的源-目标行对都没有条件成立，那么匹配的目标行将保持不变。
  * 若要使用源数据集的相应列更新目标 Delta 表的所有列，请使用 `whenMatched(...).updateAll()`。 这等效于：

    ```scala
    whenMatched(...).updateExpr(Map("col1" -> "source.col1", "col2" -> "source.col2", ...))
    ```

    针对目标 Delta 表的所有列。 因此，此操作假定源表的列与目标表的列相同，否则查询将引发分析错误。

    > [!NOTE]
    >
    > 启用自动架构迁移后，此行为将更改。 有关详细信息，请参阅[自动架构演变](#merge-schema-evolution)。

* 当源行根据匹配条件与任何目标行都不匹配时，将执行 `whenNotMatched` 子句。 这些子句具有以下语义。
  * `whenNotMatched` 子句只能具有 `insert` 操作。 新行是基于指定的列和相应的表达式生成的。 你无需指定目标表中的所有列。 对于未指定的目标列，将插入 `NULL`。

    > [!NOTE]
    >
    > 在 Databricks Runtime 6.5 及更低版本中，必须为 `INSERT` 操作提供目标表中的所有列。

  * 每个 `whenNotMatched` 子句都可以有一个可选条件。 如果存在子句条件，则仅当源条件对该行成立时才插入该行。 否则，将忽略源列。
  * 如果有多个 `whenNotMatched` 子句，则将按照指定的顺序对其进行求值（即，子句的顺序很重要）。 除最后一个之外，所有 `whenNotMatched` 子句都必须具有条件。
  * 若要使用源数据集的相应列插入目标 Delta 表的所有列，请使用 `whenNotMatched(...).insertAll()`。 这等效于：

    ```scala
    whenNotMatched(...).insertExpr(Map("col1" -> "source.col1", "col2" -> "source.col2", ...))
    ```

    针对目标 Delta 表的所有列。 因此，此操作假定源表的列与目标表的列相同，否则查询将引发分析错误。

    > [!NOTE]
    >
    > 启用自动架构迁移后，此行为将更改。 有关详细信息，请参阅[自动架构演变](#merge-schema-evolution)。

> [!IMPORTANT]
>
> 如果源数据集的多行匹配并尝试更新目标 Delta 表的相同行，则 `merge` 操作可能会失败。 根据合并的 SQL 语义，这种更新操作模棱两可，因为尚不清楚应使用哪个源行来更新匹配的目标行。 你可以预处理源表来消除出现多个匹配项的可能性。 请参阅[变更数据捕获示例](#write-change-data-into-a-delta-table) - 它对变更数据集（即源数据集）进行预处理，以仅保留每键的最新更改，然后再将更改应用到目标 Delta 表中。

> [!NOTE]
>
> 在 Databricks Runtime 7.3 中，无条件删除匹配项时允许多个匹配项（因为即使有多个匹配项，无条件删除也非常明确）。

### <a name="schema-validation"></a>架构验证

`merge` 自动验证通过插入和更新表达式生成的数据的架构是否与表的架构兼容。 它使用以下规则来确定 `merge` 操作是否兼容：

* 对于 `update` 和 `insert` 操作，指定的目标列必须存在于目标 Delta 表中。
* 对于 `updateAll` 和 `insertAll` 操作，源数据集必须具有目标 Delta 表的所有列。 源数据集可以包含额外的列，它们将被忽略。
* 对于所有操作，如果由生成目标列的表达式生成的数据类型与目标 Delta 表中的对应列不同，则 `merge` 会尝试将其转换为表中的类型。

### <a name="automatic-schema-evolution"></a><a id="automatic-schema-evolution"> </a><a id="merge-schema-evolution"> </a>自动架构演变

> [!NOTE]
>
> `merge` 中的架构演变在 Databricks Runtime 6.6 及更高版本中可用。

默认情况下，`updateAll` 和 `insertAll` 使用来自源数据集的同名列来分配目标 Delta 表中的所有列。 而忽略源数据集中与目标表中的列不匹配的任何列。 但是，在某些用例中，需要自动将源列添加到目标 Delta 表中。 若要在使用 `updateAll` 和 `insertAll`（至少其中一个）执行 `merge` 操作期间自动更新表架构，可以在运行 `merge` 操作之前将 Spark 会话配置 `spark.databricks.delta.schema.autoMerge.enabled` 设置为 `true`。

> [!NOTE]
>
> * 架构演变仅在存在 `updateAll` 或 `insertAll` 操作时（或者两者都存在时）才发生。
> * 在合并中的架构演变期间，仅顶级列（即非嵌套字段）会被更改。
> * `update` 和 `insert` 操作不能显式引用目标表中尚不存在的目标列（即使有 `updateAll` 或 `insertAll` 作为子句之一）。 请参下面的示例。

以下示例展示了在有架构演变和没有架构演变的情况下 `merge` 操作的效果。

| 列                                                                    | 查询（在 Scala 中）                                                                                                                                                                                                                                                                | 无架构演变的行为（默认值）                                                                   | 有架构演变的行为                                                                                                                                                                                                               |
|----------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 目标列：`key, value`<br><br>源列：`key, value, newValue` | `targetDeltaTable.alias("t")`<br>`  .merge(`<br>`    sourceDataFrame.alias("s"),`<br>`    "t.key = s.key")`<br>`  .whenMatched().updateAll()`<br>`  .whenNotMatched().insertAll()`<br>`  .execute()`                                                                            | 表架构保持不变；仅更新/插入列 `key`、`value`。                         | 表架构更改为 `(key, value, newValue)`。 `updateAll` 更新列`value` 和 `newValue`，而 `insertAll` 插入行 `(key, value, newValue)`。                                                                          |
| 目标列：`key, oldValue`<br><br>源列：`key, newValue`     | `targetDeltaTable.alias("t")`<br>`  .merge(`<br>`    sourceDataFrame.alias("s"),`<br>`    "t.key = s.key")`<br>`  .whenMatched().updateAll()`<br>`  .whenNotMatched().insertAll()`<br>`  .execute()`                                                                            | `updateAll` 和 `insertAll` 操作会引发错误，因为目标列 `oldValue` 不在源中。 | 表架构更改为 `(key, oldValue, newValue)`。 `updateAll` 更新列 `key` 和 `newValue`，并保留 `oldValue` 不变，而 `insertAll` 插入行 `(key, NULL, newValue)`（即，将 `oldValue` 作为 `NULL` 值插入）。 |
| 目标列：`key, oldValue`<br><br>源列：`key, newValue`     | `targetDeltaTable.alias("t")`<br>`  .merge(`<br>`    sourceDataFrame.alias("s"),`<br>`    "t.key = s.key")`<br>`  .whenMatched().update(Map(`<br>`    "newValue" -> col("s.newValue")))`<br>`  .whenNotMatched().insertAll()`<br>`  .execute()`                                 | `update` 引发错误，因为目标表中不存在列 `newValue`。                        | `update` 仍然引发错误，因为目标表中不存在列 `newValue`。                                                                                                                                                 |
| 目标列：`key, oldValue`<br><br>源列：`key, newValue`     | `targetDeltaTable.alias("t")`<br>`  .merge(`<br>`    sourceDataFrame.alias("s"),`<br>`    "t.key = s.key")`<br>`  .whenMatched().updateAll()`<br>`  .whenNotMatched().insert(Map(`<br>`    "key" -> col("s.key"),`<br>`    "newValue" -> col("s.newValue")))`<br>`  .execute()` | `insert` 引发错误，因为目标表中不存在列 `newValue`。                        | `insert` 仍然引发错误，因为目标表中不存在列 `newValue`。                                                                                                                                                      |

### <a name="performance-tuning"></a>性能调优

可以使用以下方法缩短合并所用时间：

* **缩小匹配项的搜索范围：** 默认情况下，`merge` 操作搜索整个 Delta 表以在源表中查找匹配项。 加速 `merge` 的一种方法是通过在匹配条件中添加已知约束来缩小搜索范围。 例如，假设你有一个由 `country` 和 `date` 分区的表，并且你希望使用 `merge` 更新最后一天和特定国家/地区的信息。 添加条件

  ```sql
  events.date = current_date() AND events.country = 'USA'
  ```

  将使查询更快，因为它只在相关分区中查找匹配项。 此外，该方法还有助于减少与其他并发操作发生冲突的机会。 有关详细信息，请参阅[并发控制](concurrency-control.md)。

* **压缩文件：** 如果数据存储在许多小文件中，则读取数据来搜索匹配项可能会变慢。 可以将小文件压缩为更大的文件，以提高读取吞吐量。 有关详细信息，请参阅[压缩文件](best-practices.md#compact-files)。
* **控制写入的无序分区：** `merge` 操作多次对数据进行随机排列以计算和写入更新的数据。 用于随机排列的任务的数量由 Spark 会话配置 `spark.sql.shuffle.partitions` 控制。 设置此参数不仅可以控制并行度，还可以确定输出文件的数量。 增大该值可提高并行度，但也会生成大量较小的数据文件。

* **启用优化写入：** 对于已分区表，`merge` 生成的小文件数量远大于随机分区的数量。 这是因为每个随机任务都可以在多个分区中写入多个文件，并可能成为性能瓶颈。 你可以通过启用[优化写入](optimizations/auto-optimize.md#optimized-writes)来优化这一点。

## <a name="merge-examples"></a>合并操作示例

下面是一些关于如何在不同场景中使用 `merge` 的示例。

### <a name="in-this-section"></a>本节内容：

* [写入 Delta 表时进行重复数据删除](#data-deduplication-when-writing-into-delta-tables)
* [将数据 (SCD) Type 2 操作渐变到 Delta 表](#slowly-changing-data-scd-type-2-operation-into-delta-tables)
* [将更改数据写入 Delta 表](#write-change-data-into-a-delta-table)
* [使用 foreachBatch 从流式处理查询进行更新插入](#upsert-from-streaming-queries-using-foreachbatch)

### <a name="data-deduplication-when-writing-into-delta-tables"></a><a id="data-deduplication-when-writing-into-delta-tables"> </a><a id="merge-in-dedup"> </a>写入 Delta 表时进行重复数据删除

一个常见的 ETL 用例是通过将日志附加到表中来将其收集到 Delta 表中。 但是，源通常可以生成重复的日志记录，因此需要下游重复数据删除步骤来处理它们。 通过 `merge`，你可以避免插入重复记录。

#### <a name="sql"></a>SQL

```sql
MERGE INTO logs
USING newDedupedLogs
ON logs.uniqueId = newDedupedLogs.uniqueId
WHEN NOT MATCHED
  THEN INSERT *
```

#### <a name="python"></a>Python

```python
deltaTable.alias("logs").merge(
    newDedupedLogs.alias("newDedupedLogs"),
    "logs.uniqueId = newDedupedLogs.uniqueId") \
  .whenNotMatchedInsertAll() \
  .execute()
```

#### <a name="scala"></a>Scala

```scala
deltaTable
  .as("logs")
  .merge(
    newDedupedLogs.as("newDedupedLogs"),
    "logs.uniqueId = newDedupedLogs.uniqueId")
  .whenNotMatched()
  .insertAll()
  .execute()
```

#### <a name="java"></a>Java

```java
deltaTable
  .as("logs")
  .merge(
    newDedupedLogs.as("newDedupedLogs"),
    "logs.uniqueId = newDedupedLogs.uniqueId")
  .whenNotMatched()
  .insertAll()
  .execute();
```

> [!NOTE]
>
> 包含新日志的数据集需要在其内部进行重复数据删除。 根据合并的 SQL 语义，该数据集会将新数据与表中的现有数据进行匹配并删除重复数据，但如果新数据集中存在重复数据，则将插入。 因此，在合并到表之前，请对新数据进行重复数据删除。

如果你知道几天之内可能会得到重复记录，则可以通过按日期对表进行分区，然后指定要匹配的目标表的日期范围来进一步优化查询。

#### <a name="sql"></a>SQL

```sql
MERGE INTO logs
USING newDedupedLogs
ON logs.uniqueId = newDedupedLogs.uniqueId AND logs.date > current_date() - INTERVAL 7 DAYS
WHEN NOT MATCHED AND newDedupedLogs.date > current_date() - INTERVAL 7 DAYS
  THEN INSERT *
```

#### <a name="python"></a>Python

```python
deltaTable.alias("logs").merge(
    newDedupedLogs.alias("newDedupedLogs"),
    "logs.uniqueId = newDedupedLogs.uniqueId AND logs.date > current_date() - INTERVAL 7 DAYS") \
  .whenNotMatchedInsertAll("newDedupedLogs.date > current_date() - INTERVAL 7 DAYS") \
  .execute()
```

#### <a name="scala"></a>Scala

```scala
deltaTable.as("logs").merge(
    newDedupedLogs.as("newDedupedLogs"),
    "logs.uniqueId = newDedupedLogs.uniqueId AND logs.date > current_date() - INTERVAL 7 DAYS")
  .whenNotMatched("newDedupedLogs.date > current_date() - INTERVAL 7 DAYS")
  .insertAll()
  .execute()
```

#### <a name="java"></a>Java

```java
deltaTable.as("logs").merge(
    newDedupedLogs.as("newDedupedLogs"),
    "logs.uniqueId = newDedupedLogs.uniqueId AND logs.date > current_date() - INTERVAL 7 DAYS")
  .whenNotMatched("newDedupedLogs.date > current_date() - INTERVAL 7 DAYS")
  .insertAll()
  .execute();
```

这种方法比使用前面的命令更有效，因为它仅在日志的最后 7 天而不是整个表中查找重复项。 此外，你还可以将此 insert-only merge 与结构化流式处理一起使用，以执行日志的连续重复数据删除。

* 在流式处理查询中，可以使用 `foreachBatch` 中的 merge 操作将具有重复数据删除功能的所有流数据连续写入 Delta 表。 请参阅以下[流式处理查询示例](#merge-in-streaming)，了解有关 `foreachBatch` 的详细信息。
* 在另一个流式处理查询中，你可以从此 Delta 表中连续读取重复数据删除的数据。 这是可能的，因为 insert-only merge 仅将新数据附加到 Delta 表。

> [!NOTE]
>
> Insert-only merge 经过优化后，仅在 Databricks Runtime 6.2 及更高版本中追加数据。 在 Databricks Runtime 6.1 及更低版本中，insert-only merge 操作的写入不能作为流读取。

### <a name="slowly-changing-data-scd-type-2-operation-into-delta-tables"></a><a id="merge-in-scd-type-2"> </a><a id="slowly-changing-data-scd-type-2-operation-into-delta-tables"> </a>将数据 (SCD) Type 2 操作渐变到 Delta 表

另一个常见的操作是 SCD Type 2，它维护维度表中每个键所做的所有更改的历史记录。 此类操作需要更新现有行以将键的以前值标记为旧值，并将新行作为最新值插入。 给定一个包含更新的源表和包含维数据的目标表，SCD Type 2 可以用 `merge` 表示。

下面是一个维护客户地址历史记录以及每个地址的有效日期范围的具体示例。 如果需要更新客户的地址，则必须将先前的地址标记为不是当前地址，更新其有效日期范围，然后将新地址添加为当前地址。

#### <a name="scd-type-2-using-merge-notebook"></a>使用 merge 笔记本的 SCD Type 2

[获取笔记本](../_static/notebooks/merge-in-scd-type-2.html)

### <a name="write-change-data-into-a-delta-table"></a><a id="merge-in-cdc"> </a><a id="write-change-data-into-a-delta-table"> </a>将更改数据写入 Delta 表

与 SCD 相似，另一种常见用例是将从外部数据库生成的所有数据更改应用于 Delta 表，这通常称为变更数据捕获 (CDC)。 换句话说，需要将一组应用于外部表的更新、删除和插入操作应用于 Delta 表。
可以通过使用 `merge` 来执行此操作，如下所示。

#### <a name="write-change-data-using-merge-notebook"></a>使用 MERGE 笔记本写入更改数据

[获取笔记本](../_static/notebooks/merge-in-cdc.html)

### <a name="upsert-from-streaming-queries-using-foreachbatch"></a><a id="merge-in-streaming"> </a><a id="upsert-from-streaming-queries-using-foreachbatch"> </a>使用 foreachBatch 从流式处理查询进行更新插入

可以使用 `merge` 和 `foreachBatch` 的组合（有关详细信息，请参阅 [foreachbatch](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#foreachbatch)）将复杂的更新插入操作从流式处理查询写入 Delta 表。 例如： 。

* **以更新模式写入流式处理聚合：** 这比完成模式要有效得多。
* **将数据库更改流写入 Delta 表：** 用于写入变更数据 [ 的](#merge-in-cdc) 合并查询可以在 `foreachBatch` 中使用，以连续将变更流应用于 Delta 表。
* **使用重复数据删除将流数据写入 Delta 表：** [用于重复数据删除的 insert-only merge 查询](#merge-in-dedup)可以在 `foreachBatch` 中使用自动重复数据删除功能将数据（带有重复项）连续写入到 Delta 表中。

> [!NOTE]
>
> * 请确保 `foreachBatch` 中的 `merge` 语句是幂等的，因为重启流式处理查询可以将操作多次应用于同一批数据。
> * 在 `foreachBatch` 中使用 `merge` 时，流式处理查询的输入数据速率（通过 `StreamingQueryProgress` 报告并在笔记本计算机速率图中可见）可以报告为源处生成数据的实际速率的倍数。 这是因为 `merge` 多次读取输入数据，导致输入指标倍增。 如果这是一个瓶颈，则可以在 `merge` 之前缓存批处理 DataFrame，然后在 `merge` 之后取消缓存。

#### <a name="write-streaming-aggregates-in-update-mode-using-merge-and-foreachbatch-notebook"></a>使用 merge 和 foreachBatch 笔记本在更新模式下写入流式处理聚合

[获取笔记本](../_static/notebooks/merge-in-streaming.html)