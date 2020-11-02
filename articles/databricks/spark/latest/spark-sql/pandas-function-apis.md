---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/14/2020
title: pandas 函数 API - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 pandas 函数 API。
ms.openlocfilehash: 2a525183bcc2166888d5716034a40e276a61901b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472904"
---
# <a name="pandas-function-apis"></a>pandas 函数 API

使用 pandas 函数 API，可以直接将 Python 原生函数（接受并输出 pandas 实例）应用于 PySpark 数据帧。 与 [pandas 用户定义函数](udf-python-pandas.md)类似，函数 API 也使用 [Apache Arrow](https://arrow.apache.org/) 来传输数据，并使用 pandas 来处理数据；但是，Python 类型提示在 pandas 函数 API 中是可选的。

有三种类型的 pandas 函数 API：

* 分组的映射
* 映射
* 协同分组的映射

pandas 函数 API 利用 pandas UDF 执行所使用的内部逻辑。
因此，它与 pandas UDF 具有相同的特征，例如 PyArrow、支持的 SQL 类型以及配置。

有关详细信息，请参阅博客文章：[New Pandas UDFs and Python Type Hints in the Upcoming Release of Apache Spark 3.0](https://databricks.com/blog/2020/05/20/new-pandas-udfs-and-python-type-hints-in-the-upcoming-release-of-apache-spark-3-0.html)（即将发布的 Apache Spark 3.0 中新增的 Pandas UDF 和 Python 类型提示）。

## <a name="grouped-map"></a>分组的映射

你可以通过 `groupBy().applyInPandas()` 转换已分组的数据，从而实现“拆分-应用-合并”模式。 “拆分-应用-合并”包括三个步骤：

* 使用 `DataFrame.groupBy` 将数据拆分成组。
* 对每个组应用函数。 函数的输入和输出都是 `pandas.DataFrame`。 输入数据包含每个组的所有行和列。
* 将结果组合到一个新的 `DataFrame` 中。

若要使用 `groupBy().applyInPandas()`，必须定义以下内容：

* 定义了每个组的计算的 Python 函数
* 一个 `StructType` 对象或字符串，用于定义输出 `DataFrame` 的架构

返回的 `pandas.DataFrame` 的列标签必须与所定义的输出架构中的字段名称匹配（如果指定为字符串），而如果不是字符串，则必须按位置（例如，整数索引）与字段数据类型匹配。 请参阅 [pandas.DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html#pandas.DataFrame) 来了解在构造 `pandas.DataFrame` 时如何标记列。

在应用函数之前，组的所有数据都将加载到内存中。 这可能会导致内存不足异常，特别是当组大小扭曲时。 [maxRecordsPerBatch](https://spark.apache.org/docs/latest/sql-pyspark-pandas-with-arrow.html#setting-arrow-batch-size) 的配置不应用于组，你需要确保分组的数据适合可用内存。

下面的示例演示了如何使用 `groupby().apply()` 从组中的每个值中减去平均值。

```python
df = spark.createDataFrame(
    [(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)],
    ("id", "v"))

def subtract_mean(pdf):
    # pdf is a pandas.DataFrame
    v = pdf.v
    return pdf.assign(v=v - v.mean())

df.groupby("id").applyInPandas(subtract_mean, schema="id long, v double").show()
# +---+----+
# | id|   v|
# +---+----+
# |  1|-0.5|
# |  1| 0.5|
# |  2|-3.0|
# |  2|-1.0|
# |  2| 4.0|
# +---+----+
```

有关详细用法，请参阅 [pyspark.sql.GroupedData.applyInPandas](http://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.GroupedData.applyInPandas)。

## <a name="map"></a>映射

通过 `DataFrame.mapInPandas()` 对 pandas 实例执行映射操作，以便将 `pandas.DataFrame` 的迭代器转换为 `pandas.DataFrame` 的另一个迭代器，该迭代器表示当前的 PySpark 数据帧，并以 PySpark 数据帧的形式返回结果。

基础函数接受并输出 `pandas.DataFrame` 的迭代器。
它可以返回任意长度的输出，这是相对于某些 pandas UDF（例如序列到序列 pandas UDF）而言。

下面的示例演示如何使用 `mapInPandas()`：

```python
df = spark.createDataFrame([(1, 21), (2, 30)], ("id", "age"))

def filter_func(iterator):
    for pdf in iterator:
        yield pdf[pdf.id == 1]

df.mapInPandas(filter_func, schema=df.schema).show()
# +---+---+
# | id|age|
# +---+---+
# |  1| 21|
# +---+---+
```

有关详细用法，请参阅 [pyspark.sql.DataFrame.applyInPandas](http://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame.mapInPandas)。

## <a name="cogrouped-map"></a>协同分组的映射

对于 pandas 实例的协同分组映射操作，请对要通过一个公共键协同分组的两个 PySpark `DataFrame` 使用 `DataFrame.groupby().cogroup().applyInPandas()`，然后将一个 Python 函数应用于每个协同组。 它包括以下步骤：

* 对数据进行混排，使共享一个密钥的每个数据帧的组被协同分组在一起。
* 将一个函数应用于每个协同组。 函数的输入是两个 `pandas.DataFrame`（包含一个表示密钥的可选元组）。 函数的输出是一个 `pandas.DataFrame`。
* 将所有组中的 `pandas.DataFrame` 组合成一个新的 PySpark `DataFrame`。

若要使用 `groupBy().cogroup().applyInPandas()`，必须定义以下内容：

* 一个 Python 函数，用于定义每个协同组的计算。
* 一个 `StructType` 对象或字符串，用于定义输出 PySpark `DataFrame` 的架构。

返回的 `pandas.DataFrame` 的列标签必须与所定义的输出架构中的字段名称匹配（如果指定为字符串），而如果不是字符串，则必须按位置（例如，整数索引）与字段数据类型匹配。 请参阅 [pandas.DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html#pandas.DataFrame) 来了解在构造 `pandas.DataFrame` 时如何标记列。

在应用函数之前，协同组的所有数据都将加载到内存中。 这可能会导致内存不足异常，特别是当组大小扭曲时。 [maxRecordsPerBatch](https://spark.apache.org/docs/latest/sql-pyspark-pandas-with-arrow.html#setting-arrow-batch-size) 的配置不会应用，你需要确保协同分组的数据适合可用内存。

下面的示例展示了如何使用 `groupby().cogroup().applyInPandas()` 在两个数据集之间执行 `asof join`。

```python
import pandas as pd

df1 = spark.createDataFrame(
    [(20000101, 1, 1.0), (20000101, 2, 2.0), (20000102, 1, 3.0), (20000102, 2, 4.0)],
    ("time", "id", "v1"))

df2 = spark.createDataFrame(
    [(20000101, 1, "x"), (20000101, 2, "y")],
    ("time", "id", "v2"))

def asof_join(l, r):
    return pd.merge_asof(l, r, on="time", by="id")

df1.groupby("id").cogroup(df2.groupby("id")).applyInPandas(
    asof_join, schema="time int, id int, v1 double, v2 string").show()
# +--------+---+---+---+
# |    time| id| v1| v2|
# +--------+---+---+---+
# |20000101|  1|1.0|  x|
# |20000102|  1|3.0|  x|
# |20000101|  2|2.0|  y|
# |20000102|  2|4.0|  y|
# +--------+---+---+---+
```

有关详细用法，请参阅 [pyspark.sql.PandasCogroupedOps.applyInPandas](http://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.PandasCogroupedOps.applyInPandas)。