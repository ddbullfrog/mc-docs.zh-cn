---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/14/2020
title: pandas 用户定义函数 - Azure Databricks
description: 了解如何在 Azure Databricks 中的 Python 代码中创建和使用 pandas 用户定义函数。
ms.openlocfilehash: 9660e4b8ebbfd9dd81f71090ab0f1188b1b2da93
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472744"
---
# <a name="pandas-user-defined-functions"></a><a id="pandas-udf"> </a><a id="pandas-user-defined-functions"> </a>pandas 用户定义函数

pandas 用户定义函数 (UDF) 也称为向量化 UDF，是一个用户定义函数，它使用 [Apache Arrow](https://arrow.apache.org/) 来传输数据并使用 pandas 来处理数据。 pandas UDF 允许向量化操作，与一次一行的 [Python UDF](udf-python.md) 相比，这些操作可将性能提高到 100 倍。

有关背景信息，请参阅博客文章 [New Pandas UDFs and Python Type Hints in the Upcoming Release of Apache Spark 3.0](https://databricks.com/blog/2020/05/20/new-pandas-udfs-and-python-type-hints-in-the-upcoming-release-of-apache-spark-3-0.html)（即将发布的 Apache Spark 3.0 中新增的 Pandas UDF 和 Python 类型提示）和 [Optimize conversion between PySpark and pandas DataFrames](spark-pandas.md)（优化 PySpark 与 pandas 数据帧之间的转换）。

你将使用关键字 `pandas_udf` 作为修饰器来定义 pandas UDF，并使用 [Python 类型提示](https://www.python.org/dev/peps/pep-0484/)来包装函数。
本文介绍了不同类型的 pandas UDF，并展示了如何将 pandas UDF 与类型提示配合使用。

## <a name="series-to-series-udf"></a>序列到序列 UDF

你可以使用序列到序列 pandas UDF 将标量运算矢量化。
可以将它们与 `select` 和 `withColumn` 等 API 一起使用。

Python 函数应采用 pandas 序列作为输入，并返回相同长度的 pandas 序列。你应在 Python 类型提示中指定这些。 Spark 通过以下方式运行 pandas UDF：将列拆分为批，为作为数据子集的每个批调用函数，然后将结果连接起来。

以下示例展示了如何创建一个 pandas UDF 来计算 2 个列的乘积。

```python
import pandas as pd
from pyspark.sql.functions import col, pandas_udf
from pyspark.sql.types import LongType

# Declare the function and create the UDF
def multiply_func(a: pd.Series, b: pd.Series) -> pd.Series:
    return a * b

multiply = pandas_udf(multiply_func, returnType=LongType())

# The function for a pandas_udf should be able to execute with local pandas data
x = pd.Series([1, 2, 3])
print(multiply_func(x, x))
# 0    1
# 1    4
# 2    9
# dtype: int64

# Create a Spark DataFrame, 'spark' is an existing SparkSession
df = spark.createDataFrame(pd.DataFrame(x, columns=["x"]))

# Execute function as a Spark vectorized UDF
df.select(multiply(col("x"), col("x"))).show()
# +-------------------+
# |multiply_func(x, x)|
# +-------------------+
# |                  1|
# |                  4|
# |                  9|
# +-------------------+
```

## <a name="iterator-of-series-to-iterator-of-series-udf"></a><a id="iterator-of-series-to-iterator-of-series-udf"> </a><a id="scalar-iterator-udfs"> </a>序列迭代器到序列迭代器 UDF

除了以下方面之外，迭代器 UDF 与标量 pandas UDF 相同：

* Python 函数
  * 采用批的迭代器而非单个输入批作为输入。
  * 返回输出批的迭代器，而非单个输出批。
* 迭代器中整个输出的长度应与整个输入的长度相同。
* 包装的 pandas UDF 采用单个 Spark 列作为输入。

你应将 Python 类型提示指定为 `Iterator[pandas.Series]` -> `Iterator[pandas.Series]`。

当 UDF 执行需要初始化某个状态时，此 pandas UDF 非常有用，例如，加载机器学习模型文件以将推理应用于每个输入批。

以下示例展示了如何创建具有迭代器支持的 pandas UDF。

```python
import pandas as pd
from typing import Iterator
from pyspark.sql.functions import col, pandas_udf, struct

pdf = pd.DataFrame([1, 2, 3], columns=["x"])
df = spark.createDataFrame(pdf)

# When the UDF is called with the column,
# the input to the underlying function is an iterator of pd.Series.
@pandas_udf("long")
def plus_one(batch_iter: Iterator[pd.Series]) -> Iterator[pd.Series]:
    for x in batch_iter:
        yield x + 1

df.select(plus_one(col("x"))).show()
# +-----------+
# |plus_one(x)|
# +-----------+
# |          2|
# |          3|
# |          4|
# +-----------+

# In the UDF, you can initialize some state before processing batches.
# Wrap your code with try/finally or use context managers to ensure
# the release of resources at the end.
y_bc = spark.sparkContext.broadcast(1)

@pandas_udf("long")
def plus_y(batch_iter: Iterator[pd.Series]) -> Iterator[pd.Series]:
    y = y_bc.value  # initialize states
    try:
        for x in batch_iter:
            yield x + y
    finally:
        pass  # release resources here, if any

df.select(plus_y(col("x"))).show()
# +---------+
# |plus_y(x)|
# +---------+
# |        2|
# |        3|
# |        4|
# +---------+
```

## <a name="iterator-of-multiple-series-to-iterator-of-series-udf"></a>多序列迭代器到序列迭代器 UDF

与[序列迭代器到序列迭代器 UDF](#scalar-iterator-udfs) 相比，多序列迭代器到序列迭代器 UDF 具有类似的特性和限制。 指定的函数接受批迭代器并输出批迭代器。 当 UDF 执行需要初始化某个状态时，它也很有用。

区别在于：

* 基础 Python 函数采用 pandas 序列的元组的迭代器。
* 包装的 pandas UDF 采用多个 Spark 列作为输入。

将类型提示指定为 `Iterator[Tuple[pandas.Series, ...]]` -> `Iterator[pandas.Series]`。

```python
from typing import Iterator, Tuple
import pandas as pd

from pyspark.sql.functions import col, pandas_udf, struct

pdf = pd.DataFrame([1, 2, 3], columns=["x"])
df = spark.createDataFrame(pdf)

@pandas_udf("long")
def multiply_two_cols(
        iterator: Iterator[Tuple[pd.Series, pd.Series]]) -> Iterator[pd.Series]:
    for a, b in iterator:
        yield a * b

df.select(multiply_two_cols("x", "x")).show()
# +-----------------------+
# |multiply_two_cols(x, x)|
# +-----------------------+
# |                      1|
# |                      4|
# |                      9|
# +-----------------------+
```

## <a name="series-to-scalar-udf"></a>序列到标量 UDF

序列到标量 pandas UDF 类似于 Spark 聚合函数。
序列到标量 pandas UDF 定义从一个或多个 pandas 序列到标量值的聚合，其中的每个 pandas 序列表示一个 Spark 列。
可以将序列到标量 pandas UDF 与 API（例如 `select`、`withColumn`、`groupBy.agg` 和 [pyspark.sql.Window](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.Window)）一起使用。

将类型提示表示为 `pandas.Series, ...` -> `Any`。 返回类型应当是一个基元数据类型，返回的标量可以是 Python 基元类型，例如 `int` 或 `float` 或 NumPy 数据类型（如 `numpy.int64` 或 `numpy.float64`）。 理想情况下，`Any` 应当是一个特定的标量类型。

此类型的 UDF 不支持部分聚合，每个组的所有数据都将加载到内存中。

以下示例展示了如何使用此类型的 UDF 通过 `select`、`groupBy` 和 `window` 运算来计算平均值：

```python
import pandas as pd
from pyspark.sql.functions import pandas_udf
from pyspark.sql import Window

df = spark.createDataFrame(
    [(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)],
    ("id", "v"))

# Declare the function and create the UDF
@pandas_udf("double")
def mean_udf(v: pd.Series) -> float:
    return v.mean()

df.select(mean_udf(df['v'])).show()
# +-----------+
# |mean_udf(v)|
# +-----------+
# |        4.2|
# +-----------+

df.groupby("id").agg(mean_udf(df['v'])).show()
# +---+-----------+
# | id|mean_udf(v)|
# +---+-----------+
# |  1|        1.5|
# |  2|        6.0|
# +---+-----------+

w = Window \
    .partitionBy('id') \
    .rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing)
df.withColumn('mean_v', mean_udf(df['v']).over(w)).show()
# +---+----+------+
# | id|   v|mean_v|
# +---+----+------+
# |  1| 1.0|   1.5|
# |  1| 2.0|   1.5|
# |  2| 3.0|   6.0|
# |  2| 5.0|   6.0|
# |  2|10.0|   6.0|
# +---+----+------+
```

有关详细用法，请参阅 [pyspark.sql.functions.pandas_udf](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.functions.pandas_udf)。

## <a name="usage"></a>使用情况

### <a name="setting-arrow-batch-size"></a>设置 Arrow 批大小

Spark 中的数据分区将被转换为 Arrow 记录批，这可能会暂时导致 JVM 中的内存使用率过高。 为了避免可能的内存不足异常，可以调整 Arrow 记录批的大小，方法是：将 `spark.sql.execution.arrow.maxRecordsPerBatch` 配置设置为一个用于确定每个批的最大行数的整数。 默认值为每批 10,000 条记录。 如果列数较大，则应相应地调整值。 使用此限制，每个数据分区将拆分为 1 个或多个记录批来进行处理。

### <a name="timestamp-with-time-zone-semantics"></a>包含时区语义的时间戳

Spark 在内部将时间戳存储为 UTC 值，在未指定时区的情况下引入的时间戳数据将作为本地时间转换为 UTC，并提供微秒分辨率。

在 Spark 中导出或显示时间戳数据时，将使用会话时区来本地化时间戳值。 会话时区是通过 `spark.sql.session.timeZone` 配置设置的，默认为 JVM 系统本地时区。 pandas 使用纳秒分辨率为 `datetime64[ns]` 的 `datetime64` 类型，每个列有可选的时区。

将时间戳数据从 Spark 传输到 pandas 时，会将其转换为纳秒，并将每列转换为 Spark 会话时区，然后将其本地化为该时区，这样会删除时区并将值显示为本地时间。 带时间戳列调用 `toPandas()` 或 `pandas_udf` 时，会发生这种情况。

将时间戳数据从 pandas 传输到 Spark 时，会将其转换为 UTC 微秒。 当使用 pandas 数据帧调用 `createDataFrame` 或从 pandas UDF 返回时间戳时，会发生这种情况。 这些转换会自动执行，目的是确保 Spark 具有预期格式的数据，因此你自己无需执行任何此类转换。 任何纳秒值都会被截断。

标准 UDF 将时间戳数据加载为 Python 日期/时间对象，这不同于 pandas 时间戳。 为了获得最佳性能，我们建议你在 pandas UDF 中使用时间戳时使用 pandas 时序功能。 有关详细信息，请参阅[时序/日期功能](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html)。

## <a name="example-notebook"></a>示例笔记本

以下笔记本演示了可以通过 pandas UDF 实现的性能改进：

### <a name="pandas-udfs-benchmark-notebook"></a>pandas UDF 基准笔记本

[获取笔记本](../../../_static/notebooks/pandas-udfs-benchmark.html)