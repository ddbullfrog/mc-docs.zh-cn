---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 插入 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 INSERT 语法。
ms.openlocfilehash: eceb0f9c7517a597aaca6beb0a668eaea49f70bf
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473017"
---
# <a name="insert"></a>插入

## <a name="insert-from-select-queries"></a>从 select 查询插入

```sql
INSERT INTO [TABLE] [db_name.]table_name [PARTITION part_spec] select_statement

INSERT OVERWRITE TABLE [db_name.]table_name [PARTITION part_spec] select_statement

part_spec:
  : (part_col_name1=val1 [, part_col_name2=val2, ...])
```

将数据插入 select 语句的结果表中的表或分区。 数据按顺序（列的顺序）而不是按名称插入。

**`OVERWRITE`**

覆盖表或分区中的现有数据。 否则，将追加新数据。

> [!NOTE]
>
> 在 Databricks Runtime 7.0 及更高版本中，`INSERT` 支持使用 Delta Lake 进行架构强制实施和演变。 如果列的数据类型不能安全地强制转换为 Delta 表的数据类型，则会引发运行时异常。 如果启用了[架构演变](../../../../delta/delta-batch.md#automatic-schema-update)，则新列可以作为架构的最后一列（或嵌套列）存在，以便架构得以演变。

### <a name="examples"></a>示例

```sql
-- Creates a partitioned native parquet table
CREATE TABLE data_source_tab1 (col1 INT, p1 INT, p2 INT)
  USING PARQUET PARTITIONED BY (p1, p2)

-- Appends two rows into the partition (p1 = 3, p2 = 4)
INSERT INTO data_source_tab1 PARTITION (p1 = 3, p2 = 4)
  SELECT id FROM RANGE(1, 3)

-- Overwrites the partition (p1 = 3, p2 = 4) using two new rows
INSERT OVERWRITE TABLE default.data_source_tab1 PARTITION (p1 = 3, p2 = 4)
  SELECT id FROM RANGE(3, 5)
```

## <a name="insert-values-into-tables"></a>将值插入表

```sql
INSERT INTO [TABLE] [db_name.]table_name [PARTITION part_spec] VALUES values_row [, values_row ...]

INSERT OVERWRITE TABLE [db_name.]table_name [PARTITION part_spec] VALUES values_row [, values_row ...]

values_row:
    : (val1 [, val2, ...])
```

将数据插入行值列表中的表或分区。

**`OVERWRITE`**

覆盖表或分区中的现有数据。 否则，将追加新数据。

### <a name="examples"></a>示例

```sql
-- Creates a partitioned hive serde table (using the HiveQL syntax)
CREATE TABLE hive_serde_tab1 (col1 INT, p1 INT, p2 INT)
  USING HIVE OPTIONS(fileFormat 'PARQUET') PARTITIONED BY (p1, p2)

-- Appends two rows into the partition (p1 = 3, p2 = 4)
INSERT INTO hive_serde_tab1 PARTITION (p1 = 3, p2 = 4)
  VALUES (1), (2)

-- Overwrites the partition (p1 = 3, p2 = 4) using two new rows
INSERT OVERWRITE TABLE hive_serde_tab1 PARTITION (p1 = 3, p2 = 4)
  VALUES (3), (4)
```

## <a name="dynamic-partition-inserts"></a>动态分区插入

当分区规范 `part_spec` 未完整提供时，此类插入称为动态分区插入，也称为多分区插入。 在 `part_spec` 中，分区列值是可选的。 如果未提供值，则这些列称为动态分区列；否则，它们为静态分区列。 例如，分区规范 (p1 = 3, p2, p3) 具有静态分区列 (p1) 和两个动态分区列（p2 和 p3）   。

在 `part_spec` 中，静态分区键必须位于动态分区键之前。 也就是说，所有具有常数值的分区列都需要显示在未分配常数值的其他分区列之前。

动态分区列的分区值是在执行期间确定的。 动态分区列必须在 `part_spec` 和（行值列表或 select 查询的）输入结果集中最后指定。 它们按位置而不是按名称进行解析。 因此，它们的顺序必须完全匹配。

目前，DataFrameWriter API 没有用于指定分区值的接口。 因此，其 `insertInto()` API 始终使用动态分区模式。

> [!IMPORTANT]
>
> 在动态分区模式下，输入结果集可能会导致大量的动态分区，从而生成大量的分区目录。
>
> **`OVERWRITE`**
>
> 语义因目标表的类型而异。
>
> * Hive SerDe 表：`INSERT OVERWRITE` 不会提前删除分区，只覆盖那些在运行时写入数据的分区。 这与 Apache Hive 语义匹配。 对于 Hive SerDe 表，Spark SQL 遵循与 Hive 相关的配置，包括 `hive.exec.dynamic.partition` 和 `hive.exec.dynamic.partition.mode`。
> * 本机数据源表：`INSERT OVERWRITE` 首先删除与分区规范（例如 PARTITION(a=1, b)）匹配的所有分区，然后插入所有剩余值。 可以通过将特定于会话的配置 `spark.sql.sources.partitionOverwriteMode` 更改为 `DYNAMIC`，将本机数据源表的行为更改为与 Hive SerDe 表一致。 默认模式为`STATIC`。

### <a name="examples"></a>示例

```sql
-- Create a partitioned native Parquet table
CREATE TABLE data_source_tab2 (col1 INT, p1 STRING, p2 STRING)
  USING PARQUET PARTITIONED BY (p1, p2)

-- Two partitions ('part1', 'part1') and ('part1', 'part2') are created by this dynamic insert.
-- The dynamic partition column p2 is resolved by the last column `'part' || id`
INSERT INTO data_source_tab2 PARTITION (p1 = 'part1', p2)
  SELECT id, 'part' || id FROM RANGE(1, 3)

-- A new partition ('partNew1', 'partNew2') is added by this INSERT OVERWRITE.
INSERT OVERWRITE TABLE data_source_tab2 PARTITION (p1 = 'partNew1', p2)
  VALUES (3, 'partNew2')

-- After this INSERT OVERWRITE, the two partitions ('part1', 'part1') and ('part1', 'part2') are dropped,
-- because both partitions are included by (p1 = 'part1', p2).
-- Then, two partitions ('partNew1', 'partNew2'), ('part1', 'part1') exist after this operation.
INSERT OVERWRITE TABLE data_source_tab2 PARTITION (p1 = 'part1', p2)
  VALUES (5, 'part1')

-- Create and fill a partitioned hive serde table with three partitions:
-- ('part1', 'part1'), ('part1', 'part2') and ('partNew1', 'partNew2')
CREATE TABLE hive_serde_tab2 (col1 INT, p1 STRING, p2 STRING)
  USING HIVE OPTIONS(fileFormat 'PARQUET') PARTITIONED BY (p1, p2)
INSERT INTO hive_serde_tab2 PARTITION (p1 = 'part1', p2)
  SELECT id, 'part' || id FROM RANGE(1, 3)
INSERT OVERWRITE TABLE hive_serde_tab2 PARTITION (p1 = 'partNew1', p2)
  VALUES (3, 'partNew2')

-- After this INSERT OVERWRITE, only the partitions ('part1', 'part1') is overwritten by the new value.
-- All the three partitions still exist.
INSERT OVERWRITE TABLE hive_serde_tab2 PARTITION (p1 = 'part1', p2)
  VALUES (5, 'part1')
```

## <a name="insert-values-into-directory"></a>将值插入目录

```sql
INSERT OVERWRITE [LOCAL] DIRECTORY [directory_path]
  USING data_source [OPTIONS (key1=val1, key2=val2, ...)]
  [AS] SELECT ... FROM ...
```

使用 Spark 本机格式将 `select_statement` 的查询结果插入目录 `directory_path`。 如果指定的路径存在，则会将其替换为 `select_statement` 的输出。

**`DIRECTORY`**

插入的目标目录的路径。 还可以使用键 `path` 在 `OPTIONS` 中指定该目录。 如果指定的路径存在，则会将其替换为 `select_statement` 的输出。 如果使用 `LOCAL`，则目录位于本地文件系统上。

**`USING`**

要用于插入的文件格式。 `TEXT`、`CSV`、`JSON`、`JDBC`、`PARQUET`、`ORC`、`HIVE`和 `LIBSVM` 中的一个，或 `org.apache.spark.sql.sources.DataSourceRegister` 的自定义实现的完全限定类名。

**`AS`**

用 select 语句中的输入数据填充目标目录。

### <a name="examples"></a>示例

```sql
INSERT OVERWRITE DIRECTORY
USING parquet
OPTIONS ('path' '/tmp/destination/path')
SELECT key, col1, col2 FROM source_table

INSERT OVERWRITE DIRECTORY '/tmp/destination/path'
USING json
SELECT 1 as a, 'c' as b
```

## <a name="insert-values-into-directory-with-hive-format"></a>以 Hive 格式将值插入目录

```sql
INSERT OVERWRITE [LOCAL] DIRECTORY directory_path
  [ROW FORMAT row_format] [STORED AS file_format]
  [AS] select_statement
```

使用 Hive SerDe 将 `select_statement` 的查询结果插入目录 `directory_path`。 如果指定的路径存在，则会将其替换为 `select_statement` 的输出。

> [!NOTE]
>
> 仅当启用 Hive 支持时，才支持此命令。

**`DIRECTORY`**

插入的目标目录的路径。 如果指定的路径存在，则会将其替换为 `select_statement` 的输出。 如果使用 `LOCAL`，则目录位于本地文件系统上。

**`ROW FORMAT`**

使用 `SERDE` 子句为此插入指定自定义 SerDe。 否则，请使用 `DELIMITED` 子句来使用本机 SerDe，并指定分隔符、转义字符和空字符等。

**`STORED AS`**

此插入的文件格式。 `TEXTFILE`、`SEQUENCEFILE`、`RCFILE`、`ORC`、`PARQUET` 和 `AVRO` 之一。 或者，你可以通过 `INPUTFORMAT` 和 `OUTPUTFORMAT` 指定你自己的输入和输出格式。 只能将 `TEXTFILE`、`SEQUENCEFILE` 和 `RCFILE` 用于 `ROW FORMAT SERDE`，并且只能将 `TEXTFILE` 用于 `ROW FORMAT DELIMITED`。

**`AS`**

用 select 语句中的输入数据填充目标目录。

### <a name="examples"></a>示例

```sql
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/destination/path'
STORED AS orc
SELECT * FROM source_table where key < 10
```