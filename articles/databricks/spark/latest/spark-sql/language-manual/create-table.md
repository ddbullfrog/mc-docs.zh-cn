---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/21/2020
title: 创建表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark 和 Delta Lake SQL 语言的 CREATE TABLE 语法。
ms.openlocfilehash: dec495fccd32da39a85dc101fe8c77384e89c94f
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472844"
---
# <a name="create-table"></a>创建表

## <a name="create-table-using"></a>Create Table Using

```sql
CREATE [OR REPLACE] TABLE [IF NOT EXISTS] [db_name.]table_name
  [(col_name1 col_type1 [COMMENT col_comment1], ...)]
  USING data_source
  [OPTIONS (key1=val1, key2=val2, ...)]
  [PARTITIONED BY (col_name1, col_name2, ...)]
  [CLUSTERED BY (col_name3, col_name4, ...) INTO num_buckets BUCKETS]
  [LOCATION path]
  [COMMENT table_comment]
  [TBLPROPERTIES (key1=val1, key2=val2, ...)]
  [AS select_statement]
```

使用数据源创建表。 如果数据库中已存在同名的表，则会引发异常。

**`OR REPLACE`**

如果已存在同名的表，则会使用新的配置替换该表。

> [!NOTE]
>
> 此语法在 Databricks Runtime 7.0 及更高版本中可用。 强烈建议使用 `REPLACE`，而不是删除并重新创建表。

**`IF NOT EXISTS`**

如果数据库中已存在同名的表，则不会执行任何操作。

**`USING data_source`**

将要用于表的文件格式。 `data_source` 必须是 `TEXT`、`CSV`、`JSON`、`JDBC`、`PARQUET`、`ORC`、`HIVE`、`DELTA` 或 `LIBSVM` 中的一个，或 `org.apache.spark.sql.sources.DataSourceRegister` 的自定义实现的完全限定的类名。

支持使用 `HIVE` 创建 Hive SerDe 表。 你可以使用 `OPTIONS` 子句指定 Hive 特定的 `file_format` 和 `row_format`，这是不区分大小写的字符串映射。 选项键为 `FILEFORMAT`、`INPUTFORMAT`、`OUTPUTFORMAT`、`SERDE`、`FIELDDELIM`、`ESCAPEDELIM`、`MAPKEYDELIM` 和 `LINEDELIM`。

**`OPTIONS`**

用于优化表的行为或配置 `HIVE` 表的表选项。

> [!NOTE]
>
> Delta Lake 不支持此子句。

**`PARTITIONED BY (col_name1, col_name2, ...)`**

按指定的列对创建的表进行分区。 将为每个分区创建一个目录。

**`CLUSTERED BY col_name3, col_name4, ...)`**

所创建的表中的每个分区将按指定列拆分为固定数目的 Bucket。 这通常与分区操作配合使用，以便读取和无序处理较少的数据。

**`LOCATION path`**

用于存储表数据的目录。 此子句自动隐含 `EXTERNAL`。

（Azure Databricks 上的 Delta Lake）当指定的 `LOCATION` 已包含 Delta Lake 中存储的数据时，Delta Lake 会执行以下操作：

* 如果仅指定了表名称和位置，例如：

  ```sql
  CREATE TABLE events
    USING DELTA
    LOCATION '/mnt/delta/events'
  ```

  Hive 元存储中的表会自动继承现有数据的架构、分区和表属性。 此功能可用于将数据“导入”到元存储中。

* 如果你指定了任何配置（架构、分区或表属性），则 Delta Lake 会验证指定的内容是否与现有数据的配置完全匹配。

> [!WARNING]
>
> 如果指定的配置与数据的配置并非完全匹配，则 Delta Lake 会引发一个描述差异的异常。

**`AS select_statement`**

使用来自 `SELECT` 语句的输入数据填充该表。 这不能包含列列表。

### <a name="examples"></a>示例

```sql
CREATE TABLE boxes (width INT, length INT, height INT) USING CSV

CREATE TABLE boxes
  (width INT, length INT, height INT)
  USING PARQUET
  OPTIONS ('compression'='snappy')

CREATE TABLE rectangles
  USING PARQUET
  PARTITIONED BY (width)
  CLUSTERED BY (length) INTO 8 buckets
  AS SELECT * FROM boxes

-- CREATE a HIVE SerDe table using the CREATE TABLE USING syntax.
CREATE TABLE my_table (name STRING, age INT, hair_color STRING)
  USING HIVE
  OPTIONS(
      INPUTFORMAT 'org.apache.hadoop.mapred.SequenceFileInputFormat',
      OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat',
      SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe')
  PARTITIONED BY (hair_color)
  TBLPROPERTIES ('status'='staging', 'owner'='andrew')
```

## <a name="create-table-with-hive-format"></a>使用 Hive 格式创建表

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  [(col_name1[:] col_type1 [COMMENT col_comment1], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name2[:] col_type2 [COMMENT col_comment2], ...)]
  [ROW FORMAT row_format]
  [STORED AS file_format]
  [LOCATION path]
  [TBLPROPERTIES (key1=val1, key2=val2, ...)]
  [AS select_statement]

row_format:
  : SERDE serde_cls [WITH SERDEPROPERTIES (key1=val1, key2=val2, ...)]
  | DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]]
      [COLLECTION ITEMS TERMINATED BY char]
      [MAP KEYS TERMINATED BY char]
      [LINES TERMINATED BY char]
      [NULL DEFINED AS char]

file_format:
  : TEXTFILE | SEQUENCEFILE | RCFILE | ORC | PARQUET | AVRO
  | INPUTFORMAT input_fmt OUTPUTFORMAT output_fmt
```

使用 Hive 格式创建一个表。 如果数据库中已存在同名的表，则会引发异常。 以后删除该表时，将从文件系统中删除此表中的数据。

> [!NOTE]
>
> 仅当启用了 Hive 支持时，才支持此命令。

**`EXTERNAL`**

该表使用通过 `LOCATION` 指定的自定义目录。 对表的查询将访问以前存储在目录中的现有数据。 删除 `EXTERNAL` 表时，不会从文件系统中删除其数据。 如果指定了 `LOCATION`，则隐含此标志。

**`IF NOT EXISTS`**

如果数据库中已存在同名的表，则不会执行任何操作。

**`PARTITIONED BY (col_name2[:] col_type2 [COMMENT col_comment2], ...)`**

请按指定的列对表进行分区。 这组列必须不同于非分区列的组。 不能通过 `AS select_statement` 指定已分区的列。

**`ROW FORMAT`**

使用 `SERDE` 子句为此表指定自定义 SerDe。 否则，请使用 `DELIMITED` 子句来使用本机 SerDe，并指定分隔符、转义字符和空字符等。

**`STORED AS file_format`**

指定此表的文件格式。 可用格式包括 `TEXTFILE`、`SEQUENCEFILE`、`RCFILE`、`ORC`、`PARQUET` 和 `AVRO`。 或者，你可以通过 `INPUTFORMAT` 和 `OUTPUTFORMAT` 指定你自己的输入和输出格式。 只能将格式 `TEXTFILE`、`SEQUENCEFILE` 和 `RCFILE` 用于 `ROW FORMAT SERDE`，并且只能将 `TEXTFILE` 用于 `ROW FORMAT DELIMITED`。

**`LOCATION path`**

用于存储表数据的目录。 此子句自动隐含 `EXTERNAL`。

**`AS select_statement`**

使用来自 select 语句的输入数据填充该表。 不能通过 `PARTITIONED BY` 指定此项。

## <a name="data-types"></a><a id="as"> </a><a id="data-types"> </a>数据类型

Spark SQL 支持以下数据类型：

* 数字类型
  * `ByteType`：表示 1 个字节的带符号整数。 数字范围是从 `-128` 到 `127`。
  * `ShortType`：表示 2 个字节的带符号整数。 数字范围是从 `-32768` 到 `32767`。
  * `IntegerType`：表示 4 个字节的带符号整数。 数字范围是从 `-2147483648` 到 `2147483647`。
  * `LongType`：表示 8 个字节的带符号整数。 数字范围是从 `-9223372036854775808` 到 `9223372036854775807`。
  * `FloatType`：表示 4 个字节的单精度浮点数。
  * `DoubleType`：表示 8 个字节的双精度浮点数。
  * `DecimalType`：表示任意精度的带符号十进制数字。 由 `java.math.BigDecimal` 在内部提供支持。 `BigDecimal` 由一个任意精度的非标度整数值和一个 32 位整数标度构成。
* 字符串类型：`StringType`：表示字符串值。
* 二进制类型：`BinaryType`：表示字节序列值。
* 布尔类型：`BooleanType`：表示布尔值。
* 日期/时间类型
  * `TimestampType`：表示由字段 year、month、day、hour、minute 和 second 的值构成的值，使用会话本地时区。 时间戳值表示绝对时间点。
  * `DateType`：表示由字段 year、month 和 day 的值构成的值，不包含时区。
* 复杂类型
  * `ArrayType(elementType, containsNull)`：表示由 `elementType` 类型的元素序列构成的值。 `containsNull` 用于指示 `ArrayType` 值中的元素是否可以具有 `null` 值。
  * `MapType(keyType, valueType, valueContainsNull)`：表示由一组键值对构成的值。 键的数据类型由 `keyType` 描述，而值的数据类型由 `valueType` 描述。 对于 `MapType` 值，不允许键具有 `null` 值。 `valueContainsNull` 用于指示 `MapType` 值的值是否可以具有 `null` 值。
  * `StructType(fields)`：表示多个值，其结构通过一系列 `StructField` (`fields`) 来描述。
  * `StructField(name, dataType, nullable)`：表示 `StructType` 中的字段。 字段的名称由 `name` 指示。 字段的数据类型由 `dataType` 指示。 `nullable` 用于指示这些字段的值是否可以具有 `null` 值。

下表显示了每个数据类型的类型名称和别名。

| 数据类型              | SQL 名称                                                          |
|------------------------|-------------------------------------------------------------------|
| `BooleanType`          | `BOOLEAN`                                                         |
| `ByteType`             | `BYTE`, `TINYINT`                                                 |
| `ShortType`            | `SHORT`, `SMALLINT`                                               |
| `IntegerType`          | `INT`, `INTEGER`                                                  |
| `LongType`             | `LONG`, `BIGINT`                                                  |
| `FloatType`            | `FLOAT`, `REAL`                                                   |
| `DoubleType`           | `DOUBLE`                                                          |
| `DateType`             | `DATE`                                                            |
| `TimestampType`        | `TIMESTAMP`                                                       |
| `StringType`           | `STRING`                                                          |
| `BinaryType`           | `BINARY`                                                          |
| `DecimalType`          | `DECIMAL`, `DEC`, `NUMERIC`                                       |
| `CalendarIntervalType` | `INTERVAL`                                                        |
| `ArrayType`            | `ARRAY<element_type>`                                             |
| `StructType`           | `STRUCT<field1_name: field1_type, field2_name: field2_type, ...>` |
| `MapType`              | `MAP<key_type, value_type>`                                       |

### <a name="examples"></a>示例

```sql
CREATE TABLE my_table (name STRING, age INT)

CREATE TABLE my_table (name STRING, age INT)
  COMMENT 'This table is partitioned'
  PARTITIONED BY (hair_color STRING COMMENT 'This is a column comment')
  TBLPROPERTIES ('status'='staging', 'owner'='andrew')

CREATE TABLE my_table (name STRING, age INT)
  COMMENT 'This table specifies a custom SerDe'
  ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.orc.OrcSerde'
  STORED AS
      INPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat'
      OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'

CREATE TABLE my_table (name STRING, age INT)
  COMMENT 'This table uses the CSV format'
  ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
  STORED AS TEXTFILE

CREATE TABLE your_table
  COMMENT 'This table is created with existing data'
  AS SELECT * FROM my_table

CREATE EXTERNAL TABLE IF NOT EXISTS my_table (name STRING, age INT)
  COMMENT 'This table is created with existing data'
  LOCATION 'spark-warehouse/tables/my_existing_table'
```

## <a name="create-table-like"></a>Create Table Like

```sql
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name1 LIKE [db_name.]table_name2 [LOCATION path]
```

使用现有表或视图的定义/元数据创建托管表。 创建的表始终在默认仓库位置中使用其自己的目录。

> [!NOTE]
>
> Delta Lake 不支持 `CREATE TABLE LIKE`。 请改用 `CREATE TABLE AS`。 请参阅 [AS](#as)。