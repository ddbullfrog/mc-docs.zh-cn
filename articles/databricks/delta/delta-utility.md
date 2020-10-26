---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: 表实用工具命令 - Azure Databricks
description: 了解 Delta Lake 实用工具命令。
ms.openlocfilehash: e202c2f72900ad82d2db9ea64422e8726b8109ba
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121942"
---
# <a name="table-utility-commands"></a>表实用工具命令

Delta 表支持多个实用工具命令。

## <a name="vacuum"></a><a id="delta-vacuum"> </a><a id="vacuum"> </a>Vacuum

可在表上运行 `vacuum` 命令，来删除 Delta 表中不再引用且在保留期阈值之前创建的文件。 `vacuum` 不会自动触发。 文件的默认保留期阈值为 7 天。

> [!IMPORTANT]
>
> * `vacuum` 仅删除数据文件，不删除日志文件。 检查点操作后，会自动异步删除日志文件。 日志文件的默认保留期为 30 天，可通过使用 `ALTER TABLE SET TBLPROPERTIES` SQL 方法设置的 `delta.logRetentionPeriod` 属性进行配置。 请查看[表属性](delta-batch.md#table-properties)。
> * 运行 `vacuum` 后，无法再[按时间顺序查看](delta-batch.md#deltatimetravel)在保留期之前创建的版本。

### <a name="sql"></a>SQL

```sql
VACUUM eventsTable   -- vacuum files not required by versions older than the default retention period

VACUUM '/data/events' -- vacuum files in path-based table

VACUUM delta.`/data/events/`

VACUUM delta.`/data/events/` RETAIN 100 HOURS  -- vacuum files not required by versions more than 100 hours old

VACUUM eventsTable DRY RUN    -- do dry run to get the list of files to be deleted
```

有关语法详细信息，请参阅[清空 Delta 表](../spark/latest/spark-sql/language-manual/vacuum.md)。

### <a name="python"></a>Python

> [!NOTE]
>
> 可在 Databricks Runtime 6.1 及更高版本中使用 Python API。

```python
from delta.tables import *

deltaTable = DeltaTable.forPath(spark, pathToTable)  # path-based tables, or
deltaTable = DeltaTable.forName(spark, tableName)    # Hive metastore-based tables

deltaTable.vacuum()        # vacuum files not required by versions older than the default retention period

deltaTable.vacuum(100)     # vacuum files not required by versions more than 100 hours old
```

### <a name="scala"></a>Scala

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```scala
import io.delta.tables._

val deltaTable = DeltaTable.forPath(spark, pathToTable)

deltaTable.vacuum()        // vacuum files not required by versions older than the default retention period

deltaTable.vacuum(100)     // vacuum files not required by versions more than 100 hours old
```

### <a name="java"></a>Java

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Java API。

```java
import io.delta.tables.*;
import org.apache.spark.sql.functions;

DeltaTable deltaTable = DeltaTable.forPath(spark, pathToTable);

deltaTable.vacuum();        // vacuum files not required by versions older than the default retention period

deltaTable.vacuum(100);     // vacuum files not required by versions more than 100 hours old
```

有关 Scala、Java 和 Python 语法的详细信息，请查看 [API 参考](delta-apidoc.md)。

> [!WARNING]
>
> 建议不要将保留期间隔设置为少于 7 天，因为并发读取器或写入器仍可将旧快照和未提交的文件用于表。  如果 `vacuum` 清理活动文件，则并发读取器可能会失败；更糟糕的是，当 `vacuum` 删除尚未提交的文件时，表可能会损坏。
>
> Delta Lake 具有一项安全检查，用于防止运行危险的 `vacuum` 命令。 如果确定对此表执行的操作所耗的时间均未超过计划指定的保留间隔，可通过将 Apache Spark 配置属性 `spark.databricks.delta.retentionDurationCheck.enabled` 设置为 `false` 来关闭此安全检查。 选择的时间间隔必须比最长运行并发事务长，也必须比任何流可滞后于对表的最新更新的最长时间长。

## <a name="describe-history"></a><a id="delta-history"> </a><a id="describe-history"> </a>描述历史记录

通过运行 `history` 命令，可检索每次写入 Delta 表的操作、用户和时间戳等内容的相关信息。 按时间倒序返回返回操作。 表历史记录默认保留 30 天。

### <a name="sql"></a>SQL

```sql
DESCRIBE HISTORY '/data/events/'          -- get the full history of the table

DESCRIBE HISTORY delta.`/data/events/`

DESCRIBE HISTORY '/data/events/' LIMIT 1  -- get the last operation only

DESCRIBE HISTORY eventsTable
```

有关详细信息，请查看[描述历史记录](../spark/latest/spark-sql/language-manual/describe-history.md)。

### <a name="python"></a>Python

> [!NOTE]
>
> 可在 Databricks Runtime 6.1 及更高版本中使用 Python API。

```python
from delta.tables import *

deltaTable = DeltaTable.forPath(spark, pathToTable)

fullHistoryDF = deltaTable.history()    # get the full history of the table

lastOperationDF = deltaTable.history(1) # get the last operation
```

### <a name="scala"></a>Scala

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```scala
import io.delta.tables._

val deltaTable = DeltaTable.forPath(spark, pathToTable)

val fullHistoryDF = deltaTable.history()    // get the full history of the table

val lastOperationDF = deltaTable.history(1) // get the last operation
```

### <a name="java"></a>Java

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Java API。

```java
import io.delta.tables.*;

DeltaTable deltaTable = DeltaTable.forPath(spark, pathToTable);

DataFrame fullHistoryDF = deltaTable.history();       // get the full history of the table

DataFrame lastOperationDF = deltaTable.history(1);    // fetch the last operation on the DeltaTable
```

有关 Scala/Java/Python 语法的详细信息，请查看 [API 参考](delta-apidoc.md)。

### <a name="history-schema"></a>历史记录架构

此操作的输出包含以下列。

| 列              | 类型      | 说明                                                                |
|---------------------|-----------|----------------------------------------------------------------------------|
| 版本             | long      | 通过操作生成的表版本。                                  |
| timestamp           | timestamp | 提交此版本的时间。                                           |
| userId              | 字符串    | 运行操作的用户的 ID。                                     |
| userName            | 字符串    | 运行操作的用户的姓名。                                   |
| operation           | 字符串    | 操作的名称。                                                     |
| operationParameters | map       | 操作的参数（例如谓词。）                     |
| 作业 (job)                 | struct    | 运行操作的作业的详细信息。                                 |
| 笔记本            | struct    | 运行操作的笔记本的详细信息。                      |
| clusterId           | 字符串    | 运行操作的群集的 ID。                              |
| readVersion         | long      | 读取以执行写入操作的表的版本。         |
| isolationLevel      | 字符串    | 用于此操作的隔离级别。                                   |
| isBlindAppend       | boolean   | 此操作是否追加数据。                                      |
| operationMetrics    | map       | 操作的指标（例如已修改的行数和文件数。） |
| userMetadata        | 字符串    | 用户定义的提交元数据（如果已指定）                           |

```
+-------+-------------------+------+--------+---------+--------------------+----+--------+---------+-----------+-----------------+-------------+--------------------+
|version|          timestamp|userId|userName|operation| operationParameters| job|notebook|clusterId|readVersion|   isolationLevel|isBlindAppend|    operationMetrics|
+-------+-------------------+------+--------+---------+--------------------+----+--------+---------+-----------+-----------------+-------------+--------------------+
|      5|2019-07-29 14:07:47|   ###|     ###|   DELETE|[predicate -> ["(...|null|     ###|      ###|          4|WriteSerializable|        false|[numTotalRows -> ...|
|      4|2019-07-29 14:07:41|   ###|     ###|   UPDATE|[predicate -> (id...|null|     ###|      ###|          3|WriteSerializable|        false|[numTotalRows -> ...|
|      3|2019-07-29 14:07:29|   ###|     ###|   DELETE|[predicate -> ["(...|null|     ###|      ###|          2|WriteSerializable|        false|[numTotalRows -> ...|
|      2|2019-07-29 14:06:56|   ###|     ###|   UPDATE|[predicate -> (id...|null|     ###|      ###|          1|WriteSerializable|        false|[numTotalRows -> ...|
|      1|2019-07-29 14:04:31|   ###|     ###|   DELETE|[predicate -> ["(...|null|     ###|      ###|          0|WriteSerializable|        false|[numTotalRows -> ...|
|      0|2019-07-29 14:01:40|   ###|     ###|    WRITE|[mode -> ErrorIfE...|null|     ###|      ###|       null|WriteSerializable|         true|[numFiles -> 2, n...|
+-------+-------------------+------+--------+---------+--------------------+----+--------+---------+-----------+-----------------+-------------+--------------------+
```

> [!NOTE]
>
> * 仅当使用 Databricks Runtime 6.5 或更高版本运行历史记录中的 history 命令和操作时，操作指标才可用。
> * 如果使用以下方法写入 Delta 表，则其他一些列不可用：
>   * [JDBC 或 ODBC](../integrations/bi/jdbc-odbc-bi.md)
>   * [JAR 作业](../dev-tools/api/latest/examples.md#spark-jar-job)
>   * [spark-submit 作业](../dev-tools/api/latest/examples.md#spark-submit-api-example)
>   * [使用 REST API 运行命令](../dev-tools/api/1.2/index.md#command-execution)
> * 将来添加的列将始终添加到最后一列的后面。

### <a name="operation-metrics-keys"></a>操作指标说明

`operationMetrics` 列是一个映射。

下表按操作列出了键定义。

| 操作                                                         | 指标名称                       | 说明                                                                    |
|-------------------------------------------------------------------|-----------------------------------|--------------------------------------------------------------------------------|
| WRITE、CREATE TABLE AS SELECT、REPLACE TABLE AS SELECT、COPY INTO |                                   |                                                                                |
|                                                                   | numFiles                          | 写入的文件数。                                                       |
|                                                                   | numOutputBytes                    | 已写入的内容的大小（以字节为单位）。                                         |
|                                                                   | numOutputRows                     | 写入的行数。                                                        |
| STREAMING UPDATE                                                  |                                   |                                                                                |
|                                                                   | numAddedFiles                     | 添加的文件数。                                                         |
|                                                                   | numRemovedFiles                   | 删除的文件数。                                                       |
|                                                                   | numOutputRows                     | 写入的行数。                                                        |
|                                                                   | numOutputBytes                    | 写入大小（以字节为单位）。                                                        |
| DELETE                                                            |                                   |                                                                                |
|                                                                   | numAddedFiles                     | 添加的文件数。 删除表的分区时未提供。  |
|                                                                   | numRemovedFiles                   | 删除的文件数。                                                       |
|                                                                   | numDeletedRows                    | 删除的行数。 删除表的分区时未提供。 |
|                                                                   | numCopiedRows                     | 在删除文件期间复制的行数。                        |
| TRUNCATE                                                          | numRemovedFiles                   | 删除的文件数。                                                       |
| MERGE                                                             |                                   |                                                                                |
|                                                                   | numSourceRows                     | 源数据帧中的行数。                                        |
|                                                                   | numTargetRowsInserted             | 插入到目标表的行数。                                 |
|                                                                   | numTargetRowsUpdated              | 目标表中更新的行数。                                    |
|                                                                   | numTargetRowsDeleted              | 目标表中删除的行数。                                    |
|                                                                   | numTargetRowsCopied               | 复制的目标行数。                                                  |
|                                                                   | numOutputRows                     | 写出的总行数。                                              |
|                                                                   | numTargetFilesAdded               | 添加到接收器（目标）的文件数。                                     |
|                                                                   | numTargetFilesRemoved             | 从接收器（目标）删除的文件数。                                 |
| UPDATE                                                            |                                   |                                                                                |
|                                                                   | numAddedFiles                     | 添加的文件数。                                                         |
|                                                                   | numRemovedFiles                   | 删除的文件数。                                                       |
|                                                                   | numUpdatedRows                    | 更新的行数。                                                        |
|                                                                   | numCopiedRows                     | 刚才在更新文件期间复制的行数。              |
| FSCK                                                              | numRemovedFiles                   | 删除的文件数。                                                       |
| CONVERT                                                           | numConvertedFiles                 | 已转换的 Parquet 文件数。                              |

| 操作                         | 指标名称                       | 说明                                                                                              |
|-----------------------------------|-----------------------------------|----------------------------------------------------------------------------------------------------------|
| CLONE*                            |                                   |                                                                                                          |
|                                   | sourceTableSize                   | 所克隆版本的源表的大小（以字节为单位）。                                          |
|                                   | sourceNumOfFiles                  | 源表中已克隆版本的文件数。                                        |
|                                   | numRemovedFiles                   | 目标表中删除的文件数（如果替换了先前的 Delta 表）。                    |
|                                   | removedFilesSize                  | 如果替换了先前的 Delta 表，则为目标表中删除文件的总大小（以字节为单位）。   |
|                                   | numCopiedFiles                    | 复制到新位置的文件数。 如果是浅表克隆，则为 0。                         |
|                                   | copiedFilesSize                   | 复制到新位置的文件总大小（以字节为单位）。 如果是浅表克隆，则为 0。 |
| OPTIMIZE                          |                                   |                                                                                                          |
|                                   | numAddedFiles                     | 添加的文件数。                                                                                   |
|                                   | numRemovedFiles                   | 优化的文件数。                                                                               |
|                                   | numAddedBytes                     | 优化表后添加的字节数。                                                     |
|                                   | numRemovedBytes                   | 删除的字节数。                                                                                 |
|                                   | minFileSize                       | 优化表后最小文件的大小。                                                 |
|                                   | p25FileSize                       | 优化表后第 25 个百分位文件的大小。                                          |
|                                   | p50FileSize                       | 优化表后的文件大小中值。                                                          |
|                                   | p75FileSize                       | 优化表后第 75 个百分位文件的大小。                                          |
|                                   | maxFileSize                       | 优化表后最大文件的大小。                                                  |

*需要 Databricks Runtime 7.3 或更高版本。

## <a name="describe-detail"></a><a id="delta-detail"> </a><a id="describe-detail"> </a>描述详细信息

可使用 `DESCRIBE DETAIL` 检索表的详细信息（例如文件数、数据大小）。

```sql
DESCRIBE DETAIL '/data/events/'

DESCRIBE HISTORY delta.`/data/events/`

DESCRIBE HISTORY eventsTable
```

有关详细信息，请查看[描述详细信息](../spark/latest/spark-sql/language-manual/describe-table.md)。

### <a name="detail-schema"></a>详细信息架构

此操作的输出只有一行具有以下架构。

| 列           | 类型              | 说明                                                                             |
|------------------|-------------------|-----------------------------------------------------------------------------------------|
| format           | 字符串            | 表的格式，即“delta”。                                                  |
| id               | 字符串            | 表的唯一 ID。                                                                 |
| name             | 字符串            | 在元存储中定义的表名称。                                          |
| description      | 字符串            | 表的说明。                                                               |
| location         | 字符串            | 表的位置。                                                                  |
| createdAt        | timestamp         | 表创建时间。                                                             |
| lastModified     | timestamp         | 表的上次修改时间。                                                       |
| partitionColumns | 字符串数组  | 如果表已分区，则为分区列的名称。                             |
| numFiles         | long              | 表最新版本中的文件数。                                 |
| properties       | string-string 映射 | 此表的所有属性集。                                                  |
| minReaderVersion | int               | 可读取表的读取器最低版本（由日志协议而定）。     |
| minWriterVersion | int               | 可写入表的写入器最低版本（由日志协议而定）。 |

```
+------+--------------------+------------------+-----------+--------------------+--------------------+-------------------+----------------+--------+-----------+----------+----------------+----------------+
|format|                  id|              name|description|            location|           createdAt|       lastModified|partitionColumns|numFiles|sizeInBytes|properties|minReaderVersion|minWriterVersion|
+------+--------------------+------------------+-----------+--------------------+--------------------+-------------------+----------------+--------+-----------+----------+----------------+----------------+
| delta|d31f82d2-a69f-42e...|default.deltatable|       null|file:/Users/tdas/...|2020-06-05 12:20:...|2020-06-05 12:20:20|              []|      10|      12345|        []|               1|               2|
+------+--------------------+------------------+-----------+--------------------+--------------------+-------------------+----------------+--------+-----------+----------+----------------+----------------+
```

## <a name="convert-to-delta"></a><a id="convert-to-delta"> </a><a id="generate"> </a>转换为 Delta

将现有 Parquet 表就地转换为 Delta 表。 此命令会列出目录中的所有文件，创建 Delta Lake 事务日志来跟踪这些文件，并通过读取所有 Parquet 文件的页脚来自动推断数据架构。 如果数据已分区，则必须将分区列的架构指定为 DDL 格式的字符串（即 `<column-name1> <type>, <column-name2> <type>, ...`）。

### <a name="sql"></a>SQL

```sql
-- Convert unpartitioned parquet table at path '<path-to-table>'
CONVERT TO DELTA parquet.`<path-to-table>`

-- Convert partitioned Parquet table at path '<path-to-table>' and partitioned by integer columns named 'part' and 'part2'
CONVERT TO DELTA parquet.`<path-to-table>` PARTITIONED BY (part int, part2 int)
```

有关详细信息，请参阅[转换为 Delta（Azure Databricks 上的 Delta Lake）](../spark/latest/spark-sql/language-manual/convert-to-delta.md)命令。

### <a name="python"></a>Python

> [!NOTE]
>
> 可在 Databricks Runtime 6.1 及更高版本中使用 Python API。

```python
from delta.tables import *

# Convert unpartitioned parquet table at path '<path-to-table>'
deltaTable = DeltaTable.convertToDelta(spark, "parquet.`<path-to-table>`")

# Convert partitioned parquet table at path '<path-to-table>' and partitioned by integer column named 'part'
partitionedDeltaTable = DeltaTable.convertToDelta(spark, "parquet.`<path-to-table>`", "part int")
```

### <a name="scala"></a>Scala

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```scala
import io.delta.tables._

// Convert unpartitioned Parquet table at path '<path-to-table>'
val deltaTable = DeltaTable.convertToDelta(spark, "parquet.`<path-to-table>`")

// Convert partitioned Parquet table at path '<path-to-table>' and partitioned by integer columns named 'part' and 'part2'
val partitionedDeltaTable = DeltaTable.convertToDelta(spark, "parquet.`<path-to-table>`", "part int, part2 int")
```

### <a name="java"></a>Java

> [!NOTE]
>
> 可在 Databricks Runtime 6.0 及更高版本中使用 Scala API。

```java
import io.delta.tables.*;

// Convert unpartitioned parquet table at path '<path-to-table>'
DeltaTable deltaTable = DeltaTable.convertToDelta(spark, "parquet.`<path-to-table>`");

// Convert partitioned Parquet table at path '<path-to-table>' and partitioned by integer columns named 'part' and 'part2'
DeltaTable deltaTable = DeltaTable.convertToDelta(spark, "parquet.`<path-to-table>`", "part int, part2 int");
```

> [!NOTE]
>
> Delta Lake 未跟踪的文件均不可见，运行 `vacuum` 时可将其删除。 在转换过程中，请勿更新或追加数据文件。 转换表后，请确保通过 Delta Lake 执行所有写入。

## <a name="convert-a-delta-table-to-a-parquet-table"></a>将 Delta 表转换为 Parquet 表

可按照以下步骤将 Delta 表轻松地重新转换为 Parquet 表：

1. 如果执行了可更改数据文件的 Delta Lake 操作（例如 `delete` 或 `merge`），请运行 [vacuum](#delta-vacuum) 并将保留期设为 0 小时，从而删除表的最新版本中未包含的所有数据文件。
2. 删除表目录中的 `_delta_log` 目录。

## <a name="clone-a-delta-table"></a><a id="clone-a-delta-table"> </a><a id="clone-delta-table"> </a>克隆 Delta 表

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../release-notes/release-types.md)提供。

> [!NOTE]
>
> 可在 Databricks Runtime 7.2 及更高版本中使用。

可使用 `clone` 命令在特定版本创建现有 Delta 表的副本。 克隆可以是深层克隆，也可是浅表克隆。

* 除了克隆现有表的元数据，深层克隆还会将源表数据复制到克隆目标。 此外，它还会克隆流元数据，使写入 Delta 表的流可在源表上停止，并在克隆的目标位置（即停止位置）继续进行克隆。
* 浅表克隆不会将数据文件复制到克隆目标。 表元数据等效于源。 创建这些克隆的成本较低。

对深层克隆或浅表克隆所做的任何更改都只影响克隆本身，不会影响源表。

克隆的元数据包括：架构、分区信息、不变性、为 Null 性。 深层克隆还会克隆流和 [COPY INTO](../spark/latest/spark-sql/language-manual/copy-into.md) 元数据（浅表克隆不克隆这些内容）。 未克隆的两种元数据是表说明和[用户定义的提交元数据](delta-batch.md#set-user-defined-commit-metadata)这。

> [!IMPORTANT]
>
> * 浅表克隆会引用源目录中的数据文件。 如果在源表上运行 `vacuum`，客户端将无法再读取引用的数据文件，并且将引发 `FileNotFoundException`。 在这种情况下，在浅表克隆上运行带有 replace 的克隆将修复该克隆。 如果此情况经常发生，请考虑使用深层克隆，而不是依赖于源表。
> * 深层克隆不依赖进行克隆的源，但由于深层克隆会复制数据和元数据，因此创建成本很高。
> * 使用 `replace` 克隆到已在该路径具有表的目标时，如果该路径不存在，会创建一个 Delta 日志。 可运行 `vacuum` 来清理任何现有数据。 如果现有表是 Delta 表，则会在现有 Delta 表上创建新的提交，其中包括源表中的新元数据和新数据。
> * 克隆表与 `Create Table As Select` 或 `CTAS` 不同。 除数据外，克隆还会复制源表的元数据。 而且，克隆的语法更为简单：无需指定分区、格式、不变性和为 Null 性等，因为它们取自源表。
> * 克隆的表具有与其源表无关的历史记录。 在克隆的表上按时间顺序查询时，这些查询使用的输入与它们在其源表上查询时使用的不同。

### <a name="sql"></a>SQL

```sql
 CREATE TABLE delta.`/data/target/` CLONE delta.`/data/source/` -- Create a deep clone of /data/source at /data/target

 CREATE OR REPLACE TABLE db.target_table CLONE db.source_table -- Replace the target

 CREATE TABLE IF NOT EXISTS TABLE delta.`/data/target/` CLONE db.source_table -- No-op if the target table exists

 CREATE TABLE db.target_table SHALLOW CLONE delta.`/data/source`

 CREATE TABLE db.target_table SHALLOW CLONE delta.`/data/source` VERSION AS OF version

 CREATE TABLE db.target_table SHALLOW CLONE delta.`/data/source` TIMESTAMP AS OF timestamp_expression -- timestamp can be like “2019-01-01” or like date_sub(current_date(), 1)
```

### <a name="python"></a>Python

```python
 from delta.tables import *

 deltaTable = DeltaTable.forPath(spark, pathToTable)  # path-based tables, or
 deltaTable = DeltaTable.forName(spark, tableName)    # Hive metastore-based tables

 deltaTable.clone(target, isShallow, replace) # clone the source at latest version

 deltaTable.cloneAtVersion(version, target, isShallow, replace) # clone the source at a specific version

# clone the source at a specific timestamp such as timestamp=“2019-01-01”
 deltaTable.cloneAtTimestamp(timestamp, target, isShallow, replace)
```

### <a name="scala"></a>Scala

```scala
 import io.delta.tables._

 val deltaTable = DeltaTable.forPath(spark, pathToTable)
 val deltaTable = DeltaTable.forName(spark, tableName)

 deltaTable.clone(target, isShallow, replace) // clone the source at latest version

 deltaTable.cloneAtVersion(version, target, isShallow, replace) // clone the source at a specific version

 deltaTable.cloneAtTimestamp(timestamp, target, isShallow, replace) // clone the source at a specific timestamp
```

### <a name="java"></a>Java

```java
 import io.delta.tables.*;

 DeltaTable deltaTable = DeltaTable.forPath(spark, pathToTable);
 DeltaTable deltaTable = DeltaTable.forName(spark, tableName);

 deltaTable.clone(target, isShallow, replace) // clone the source at latest version

 deltaTable.cloneAtVersion(version, target, isShallow, replace) // clone the source at a specific version

 deltaTable.cloneAtTimestamp(timestamp, target, isShallow, replace) // clone the source at a specific timestamp
```

有关语法详细信息，请参阅[克隆（Azure Databricks 上的 Delta Lake）](../spark/latest/spark-sql/language-manual/clone.md)。

### <a name="permissions"></a>权限

必须分别为 Azure Databricks 表 ACL 和云提供商配置 `CLONE` 所需的权限。

#### <a name="table-acls"></a>表 ACL

深层克隆和浅表克隆都需要以下权限：

* 对源表的 `SELECT` 权限。
* 如果使用 `CLONE` 创建新表，请对创建表的数据库具有 `CREATE` 权限。
* 如果使用 `CLONE` 替换表，则必须对表具有 `MODIFY` 权限。

#### <a name="cloud-permissions"></a>云权限

如果已创建深层克隆，则任何读取深层克隆的用户都必须对该克隆的目录具有读取访问权限。 若要更改克隆，用户必须对克隆的目录具有写入访问权限。

如果已创建浅表克隆，则任何读取该浅表克隆的用户都需要权限才能读取原始表中的文件，因为数据文件保留在源表中，并包含浅表克隆以及克隆的目录。 若要更改克隆，用户需要对克隆的目录具有写入访问权限。

### <a name="clone-use-cases"></a>克隆用例

* 数据存档：数据保存的时间可能需要比按时间顺序查看可实现的或者灾难恢复所需的时间长。 在这些情况下，你可创建深层克隆，保留表在某个时间点的状态以供存档。 还可通过增量存档来保留源表的持续更新状态以进行灾难恢复。

  ```sql
  -- Every month run
  CREATE OR REPLACE TABLE delta.`/some/archive/path` CLONE my_prod_table
  ```

* 机器学习流重现：在进行机器学习时，你可能希望将已训练 ML 模型的表的特定版本进行存档。 可使用此存档数据集测试将来的模型。

  ```sql
      -- Trained model on version 15 of Delta table
  CREATE TABLE delta.`/model/dataset` CLONE entire_dataset VERSION AS OF 15
  ```

* 在生产表上进行短期试验：为了在不损坏表的情况下测试生产表中的工作流，可轻松创建一个浅表克隆。 这样，就可在包含所有生产数据的克隆表上运行任意工作流，而不会影响任何生产工作负载。

  ```sql
  -- Perform shallow clone
  CREATE OR REPLACE TABLE my_test SHALLOW CLONE my_prod_table;

  UPDATE my_test WHERE user_id is null SET invalid=true;
  -- Run a bunch of validations. Once happy:

  -- This should leverage the update information in the clone to prune to only
  -- changed files in the clone if possible
  MERGE INTO my_prod_table
  USING my_test
  ON my_test.user_id <=> my_prod_table.user_id
  WHEN MATCHED AND my_test.user_id is null THEN UPDATE *;

  DROP TABLE my_test;
  ```

* 数据共享：单个组织内的其他业务部门可能也需要访问上述数据，但可能不需要最新更新。 可为不同的业务部门提供不同权限的克隆，而不是直接授予对源表的访问权限。 克隆的性能比简单视图的性能更高。

  ```sql
  -- Perform deep clone
  CREATE OR REPLACE TABLE shared_table CLONE my_prod_table;

  -- Grant other users access to the shared table
  GRANT SELECT ON shared_table TO `<user-name>@<user-domain>.com`;
  ```