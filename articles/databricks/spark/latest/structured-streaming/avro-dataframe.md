---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 读取和写入流 Avro 数据 - Azure Databricks
description: 了解如何在 Apache Kafka 中使用 Apache Avro 数据作为源和接收器，以便在 Azure Databricks 中流式处理数据。
ms.openlocfilehash: 068b6c64e8075123dfb3b06d3462420d329094b9
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472986"
---
# <a name="read-and-write-streaming-avro-data"></a>读取和写入流 Avro 数据

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

[Apache Avro](https://avro.apache.org/) 是流式处理领域中常用的数据序列化系统。 典型的解决方案是将数据以 Avro 格式放在 Apache Kafka 中，将元数据放在 [Confluent 架构注册表](https://docs.confluent.io/current/schema-registry/docs/index.html)中，然后使用同时连接到 Kafka 和架构注册表的流式处理框架运行查询。

Azure Databricks 支持 `from_avro` 和 `to_avro` [函数](https://spark.apache.org/docs/latest/sql-data-sources-avro.html#to_avro-and-from_avro)，允许使用 Kafka 中的 Avro 数据和架构注册表中的元数据来构建流式处理管道。 函数 `to_avro` 将列编码为 Avro 格式的二进制数据，而 `from_avro` 将 Avro 二进制数据解码为列。 这两个函数都将一个列转换为另一个列，而输入/输出 SQL 数据类型可以是复杂类型或基元类型。

> [!NOTE]
>
> `from_avro` 和 `to_avro` 函数：
>
> * 在 [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=from_avro#module-pyspark.sql.avro.functions)、Scala 和 Java 中可用。
> * 可以在批中和流式处理查询中传递到 SQL 函数。

另请参阅 [Avro 文件数据源](../../../data/data-sources/read-avro.md)。

## <a name="basic-example"></a>基本示例

与 [from_json](../spark-sql/language-manual/functions.md#from_json) 和 [to_json](../spark-sql/language-manual/functions.md#to_json) 类似，可以将 `from_avro` 和 `to_avro` 用于任何二进制列，但必须手动指定 Avro 架构。

```scala
import org.apache.spark.sql.avro.functions._
import org.apache.avro.SchemaBuilder

// When reading the key and value of a Kafka topic, decode the
// binary (Avro) data into structured data.
// The schema of the resulting DataFrame is: <key: string, value: int>
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", servers)
  .option("subscribe", "t")
  .load()
  .select(
    from_avro($"key", SchemaBuilder.builder().stringType()).as("key"),
    from_avro($"value", SchemaBuilder.builder().intType()).as("value"))

// Convert structured data to binary from string (key column) and
// int (value column) and save to a Kafka topic.
dataDF
  .select(
    to_avro($"key").as("key"),
    to_avro($"value").as("value"))
  .writeStream
  .format("kafka")
  .option("kafka.bootstrap.servers", servers)
  .option("article", "t")
  .save()
```

## <a name="jsonformatschema-example"></a>jsonFormatSchema 示例

还可以 JSON 字符串的形式指定架构。 例如，如果 `/tmp/user.avsc` 为：

```json
{
  "namespace": "example.avro",
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "favorite_color", "type": ["string", "null"]}
  ]
}
```

可以创建一个 JSON 字符串：

```python
from pyspark.sql.avro.functions import from_avro, to_avro

jsonFormatSchema = open("/tmp/user.avsc", "r").read()
```

然后在 `from_avro` 中使用该架构：

```python
# 1. Decode the Avro data into a struct.
# 2. Filter by column "favorite_color".
# 3. Encode the column "name" in Avro format.

output = df\
  .select(from_avro("value", jsonFormatSchema).alias("user"))\
  .where('user.favorite_color == "red"')\
  .select(to_avro("user.name").alias("value"))
```

## <a name="example-with-schema-registry"></a>使用架构注册表的示例

如果群集具有架构注册表服务，则 `from_avro` 可以使用该服务，这样你就无需手动指定 Avro 架构。

> [!NOTE]
>
> 与架构注册表的集成仅适用于 Scala 和 Java。

```scala
import org.apache.spark.sql.avro.functions._

// Read a Kafka topic "t", assuming the key and value are already
// registered in Schema Registry as subjects "t-key" and "t-value" of type
// string and int. The binary key and value columns are turned into string
// and int type with Avro and Schema Registry. The schema of the resulting DataFrame
// is: <key: string, value: int>.
val schemaRegistryAddr = "https://myhost:8081"
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", servers)
  .option("subscribe", "t")
  .load()
  .select(
    from_avro($"key", "t-key", schemaRegistryAddr).as("key"),
    from_avro($"value", "t-value", schemaRegistryAddr).as("value"))
```

对于 `to_avro`，默认输出 Avro 架构可能与架构注册表服务中目标使用者的架构不匹配，原因如下：

* 从 Spark SQL 类型到 Avro 架构的映射不是一对一。 请参阅 [Spark SQL -> Avro 转换支持的类型](../../../data/data-sources/read-avro.md#supported-types-for-spark-sql---avro-conversion)。
* 如果转换后的输出 Avro 模式是记录类型，则记录名称为 `topLevelRecord`，默认情况下没有命名空间。

如果 `to_avro` 的默认输出架构与目标使用者的架构匹配，则可执行以下代码：

```scala
// The converted data is saved to Kafka as a Kafka topic "t".
dataDF
  .select(
    to_avro($"key", lit("t-key"), schemaRegistryAddr).as("key"),
    to_avro($"value", lit("t-value"), schemaRegistryAddr).as("value"))
.writeStream
.format("kafka")
.option("kafka.bootstrap.servers", servers)
.option("article", "t")
.save()
```

否则，必须在 `to_avro` 函数中提供目标使用者的架构：

```scala
// The Avro schema of subject "t-value" in JSON string format.
val avroSchema = ...
// The converted data is saved to Kafka as a Kafka topic "t".
dataDF
  .select(
    to_avro($"key", lit("t-key"), schemaRegistryAddr).as("key"),
    to_avro($"value", lit("t-value"), schemaRegistryAddr, avroSchema).as("value"))
.writeStream
.format("kafka")
.option("kafka.bootstrap.servers", servers)
.option("article", "t")
.save()
```