---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: Avro 文件 - Azure Databricks
description: 了解如何使用 Azure Databricks 在 Avro 文件中读取和写入数据。
ms.openlocfilehash: 0d00ed74805629e187ecfcdee80b65d2fddc59c5
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121845"
---
# <a name="avro-file"></a>Avro 文件

[Apache Avro](https://avro.apache.org/) 是一个数据序列化系统。 Avro 提供：

* 丰富的数据结构。
* 精简、快速的二进制数据格式。
* 一个容器文件，用于存储持久性数据。
* 远程过程调用 (RPC)。
* 与动态语言的简单集成。 不需要进行代码生成便可读取或写入数据文件，也不需要使用或实现 RPC 协议。 代码生成作为一种可选的优化，只值得为静态类型的语言实现。

[Avro 数据源](https://spark.apache.org/docs/latest/sql-data-sources-avro.html)支持：

* 架构转换：Apache Spark SQL 与 Avro 记录之间的自动转换。
* 分区：无需任何额外配置即可轻松读取和写入分区的数据。
* 压缩：将 Avro 写入磁盘时要使用的压缩。 支持的类型包括 `uncompressed`、`snappy` 和 `deflate`。 还可以指定 deflate 级别。
* 记录名称：通过使用 `recordName` 和 `recordNamespace` 传递参数的映射来记录名称和命名空间。

另请参阅[读取和写入流 Avro 数据](../../spark/latest/structured-streaming/avro-dataframe.md)。

## <a name="configuration"></a>配置

你可以使用各种配置参数更改 Avro 数据源的行为。

若要在读取时忽略没有 `.avro` 扩展名的文件，可以在 Hadoop 配置中设置参数 `avro.mapred.ignore.inputs.without.extension`。 默认值为 `false`。

```scala
spark.
  .sparkContext
  .hadoopConfiguration
  .set("avro.mapred.ignore.inputs.without.extension", "true")
```

若要配置写入时的压缩，请设置以下 Spark 属性：

* 压缩编解码器：`spark.sql.avro.compression.codec`。 支持的编解码器为 `snappy` 和 `deflate`。 默认编解码器为 `snappy`。
* 如果压缩编解码器为 `deflate`，则可通过 `spark.sql.avro.deflate.level` 设置压缩级别。 默认级别为 `-1`。

可以在群集 [Spark 配置](../../clusters/configure.md#spark-config)中或在运行时使用 `spark.conf.set()` 设置这些属性。 例如： 。

```scala
spark.conf.set("spark.sql.avro.compression.codec", "deflate")
spark.conf.set("spark.sql.avro.deflate.level", "5")
```

## <a name="supported-types-for-avro---spark-sql-conversion"></a>Avro -> Spark SQL 转换支持的类型

此库支持读取所有 Avro 类型。 它使用下述从 Avro 类型到 Spark SQL 类型的映射：

| Avro 类型       | Spark SQL 类型                   |
|-----------------|----------------------------------|
| boolean         | BooleanType                      |
| int             | IntegerType                      |
| long            | LongType                         |
| FLOAT           | FloatType                        |
| double          | DoubleType                       |
| 字节           | BinaryType                       |
| 字符串          | StringType                       |
| 记录 (record)          | StructType                       |
| enum            | StringType                       |
| array           | ArrayType                        |
| map             | MapType                          |
| fixed           | BinaryType                       |
| union           | 请参阅[联合类型](#union-types)。 |

### <a name="union-types"></a>联合类型

Avro 数据源支持读取 `union` 类型。 Avro 将以下三种类型视为 `union` 类型：

* `union(int, long)` 映射到 `LongType`。
* `union(float, double)` 映射到 `DoubleType`。
* `union(something, null)`，其中 `something` 是任何受支持的 Avro 类型。 这会映射到与 `something` 相同的 Spark SQL 类型，`nullable` 设置为 `true`。

所有其他 `union` 类型都是复杂类型。 它们将映射到 `StructType`，其中的字段名称是 `member0`、`member1` 等，与 `union` 的成员保持一致。 这与在 Avro 和 Parquet 之间进行转换时的行为一致。

### <a name="logical-types"></a>逻辑类型

Avro 数据源支持读取以下 [Avro 逻辑类型](https://avro.apache.org/docs/1.8.2/spec.html#Logical+Types)：

| Avro 逻辑类型     | Avro 类型     | Spark SQL 类型     |
|-----------------------|---------------|--------------------|
| date                  | int           | DateType           |
| timestamp-millis      | long          | TimestampType      |
| timestamp-micros      | long          | TimestampType      |
| Decimal               | fixed         | DecimalType        |
| Decimal               | 字节         | DecimalType        |

> [!NOTE]
>
> Avro 数据源会忽略 Avro 文件中提供的文档、别名和其他属性。

## <a name="supported-types-for-spark-sql---avro-conversion"></a>Spark SQL -> Avro 转换支持的类型

此库支持将所有 Spark SQL 类型写入 Avro。 对于大多数类型，从 Spark 类型到 Avro 类型的映射直接了当（例如 `IntegerType` 转换为 `int`）；下面列出了几种特殊情况：

| Spark SQL 类型       | Avro 类型       | Avro 逻辑类型     |
|----------------------|-----------------|-----------------------|
| ByteType             | int             |                       |
| ShortType            | int             |                       |
| BinaryType           | 字节           |                       |
| DecimalType          | fixed           | Decimal               |
| TimestampType        | long            | timestamp-micros      |
| DateType             | int             | date                  |

你还可以通过选项 `avroSchema` 指定整个输出 Avro 架构，使 Spark SQL 类型可以转换为其他 Avro 类型。
以下转换默认情况下不会应用，需要用户指定的 Avro 架构：

| Spark SQL 类型       | Avro 类型       | Avro 逻辑类型     |
|----------------------|-----------------|-----------------------|
| ByteType             | fixed           |                       |
| StringType           | enum            |                       |
| DecimalType          | 字节           | Decimal               |
| TimestampType        | long            | timestamp-millis      |

## <a name="examples"></a>示例

这些示例使用 [episodes.avro](../../_static/examples/episodes.avro) 文件。

### <a name="scala"></a>Scala

```scala
// The Avro records are converted to Spark types, filtered, and
// then written back out as Avro records

val df = spark.read.format("avro").load("/tmp/episodes.avro")
df.filter("doctor > 5").write.format("avro").save("/tmp/output")
```

此示例演示了一个自定义 Avro 架构：

```scala
import org.apache.avro.Schema

val schema = new Schema.Parser().parse(new File("episode.avsc"))

spark
  .read
  .format("avro")
  .option("avroSchema", schema.toString)
  .load("/tmp/episodes.avro")
  .show()
```

此示例演示了 Avro 压缩选项：

```scala
// configuration to use deflate compression
spark.conf.set("spark.sql.avro.compression.codec", "deflate")
spark.conf.set("spark.sql.avro.deflate.level", "5")

val df = spark.read.format("avro").load("/tmp/episodes.avro")

// writes out compressed Avro records
df.write.format("avro").save("/tmp/output")
```

此示例演示了分区的 Avro 记录：

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder().master("local").getOrCreate()

val df = spark.createDataFrame(
  Seq(
    (2012, 8, "Batman", 9.8),
    (2012, 8, "Hero", 8.7),
    (2012, 7, "Robot", 5.5),
    (2011, 7, "Git", 2.0))
  ).toDF("year", "month", "title", "rating")

df.toDF.write.format("avro").partitionBy("year", "month").save("/tmp/output")
```

此示例演示了记录名称和命名空间：

```scala
val df = spark.read.format("avro").load("/tmp/episodes.avro")

val name = "AvroTest"
val namespace = "org.foo"
val parameters = Map("recordName" -> name, "recordNamespace" -> namespace)

df.write.options(parameters).format("avro").save("/tmp/output")
```

### <a name="python"></a>Python

```python
# Create a DataFrame from a specified directory
df = spark.read.format("avro").load("/tmp/episodes.avro")

#  Saves the subset of the Avro records read in
subset = df.where("doctor > 5")
subset.write.format("avro").save("/tmp/output")
```

### <a name="sql"></a>SQL

若要采用 SQL 来查询 Avro 数据，请将数据文件注册为表或临时视图：

```sql
CREATE TEMPORARY VIEW episodes
USING avro
OPTIONS (path "/tmp/episodes.avro")

SELECT * from episodes
```

## <a name="notebook"></a>笔记本

以下笔记本演示了如何读取和写入 Avro 文件。

### <a name="read-and-write-avro-files-notebook"></a>读取和写入 Avro 文件笔记本

[获取笔记本](../../_static/notebooks/read-avro-files.html)