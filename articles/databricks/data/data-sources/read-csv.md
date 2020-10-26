---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/22/2020
title: CSV 文件 - Azure Databricks
description: 了解如何使用 Azure Databricks 读取数据并将数据写入 CSV 文件。
ms.openlocfilehash: 4f45774ac16e64790e20720bfae9e289a6f789ee
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121919"
---
# <a name="csv-file"></a><a id="csv"> </a><a id="csv-file"> </a>CSV 文件

## <a name="options"></a>选项

有关支持的读取和写入选项，请参阅以下 Apache Spark 参考文章。

* 读取
  * [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=dataframereader#pyspark.sql.DataFrameReader.csv)
  * [Scala](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/DataFrameReader.html#csv(path:String):Unit)
* 写入
  * [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=dataframereader#pyspark.sql.DataFrameWriter.csv)
  * [Scala](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/DataFrameWriter.html#csv(path:String):Unit)

## <a name="examples"></a>示例

这些示例使用[钻石数据集](../databricks-datasets.md#databricks-datasets)。 指定数据集的路径以及所需的任何选项。

### <a name="in-this-section"></a>本节内容：

* [用任何语言读取文件](#read-file-in-any-language)
* [指定架构](#specify-schema)
* [验证数据的正确性](#verify-correctness-of-the-data)
* [读取列子集的错误](#pitfalls-of-reading-a-subset-of-columns)

### <a name="read-file-in-any-language"></a>用任何语言读取文件

此笔记本演示如何使用 Scala、R、Python 和 SQL 读取文件、显示示例数据和打印数据架构。

#### <a name="read-csv-files-notebook"></a>读取 CSV 文件的笔记本

[获取笔记本](../../_static/notebooks/read-csv-files.html)

### <a name="specify-schema"></a>指定架构

当 CSV 文件的架构已知时，可以用 `schema` 选项向 CSV 读取器指定所需的架构。

#### <a name="read-csv-files-with-schema-notebook"></a>使用架构读取 CSV 文件的笔记本

[获取笔记本](../../_static/notebooks/read-csv-schema.html)

### <a name="verify-correctness-of-the-data"></a>验证数据的正确性

使用指定的架构读取 CSV 文件时，文件中的数据可能与架构不匹配。 例如，包含城市名称的字段将不会分析为整数。 结果取决于分析程序运行的模式：

* `PERMISSIVE`（默认）：对于无法正确分析的字段，插入 null
* `DROPMALFORMED`：删除包含无法分析的字段的行
* `FAILFAST`：如果发现任何格式错误的数据，则中止读取

若要设置模式，请使用 `mode` 选项。

```scala
val diamonds_with_wrong_schema_drop_malformed = sqlContext.read.format("csv").option("mode", "PERMISSIVE")
```

在 `PERMISSIVE` 模式下，可以检查无法正确分析的行。 为此，可以将 `_corrupt_record` 列添加到架构。

#### <a name="find-malformed-rows-notebook"></a>查找格式错误的行的笔记本

[获取笔记本](../../_static/notebooks/read-csv-corrupt-record.html)

### <a name="pitfalls-of-reading-a-subset-of-columns"></a>读取列子集的错误

CSV 分析程序的行为取决于读取的列集。 如果指定的架构不正确，则结果会有很大差异，具体取决于访问的列的子集。 以下笔记本提供最常见的错误。

#### <a name="caveats-of-reading-a-subset-of-columns-of-a-csv-file-notebook"></a>读取 CSV 文件的列子集的注意事项的笔记本

[获取笔记本](../../_static/notebooks/read-csv-column-subset.html)