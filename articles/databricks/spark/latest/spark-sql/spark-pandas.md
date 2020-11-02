---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/14/2020
title: 优化 PySpark 与 Pandas 数据帧之间的转换 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Arrow 在 Apache Spark 数据帧与 Pandas 数据帧之间进行转换。
ms.openlocfilehash: 056176caa11b347151b84b6f209ac55231a52876
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473071"
---
# <a name="optimize-conversion-between-pyspark-and-pandas-dataframes"></a>优化 PySpark 与 Pandas 数据帧之间的转换

[Apache Arrow](https://arrow.apache.org/) 是一种内存中纵栏式数据格式，在 Apache Spark 中用于在 JVM 和 Python 进程之间高效传输数据。 这对于处理 Pandas 和 NumPy 数据的 Python 开发人员非常有利。 但是，其使用并不是自动的，需要对配置或代码进行一些小改动，才能充分利用并确保兼容性。

## <a name="pyarrow-versions"></a>PyArrow 版本

PyArrow 安装在 Databricks Runtime 中。 有关每个 Databricks Runtime 版本中可用的 PyArrow 版本的信息，请参阅 [Databricks Runtime 发行说明](../../../release-notes/runtime/index.md)。

## <a name="supported-sql-types"></a>支持的 SQL 类型

基于 Arrow 的转换支持除 `MapType`、`TimestampType` 的 `ArrayType` 和嵌套的 `StructType` 外的所有 Spark SQL 数据类型。 `StructType` 表示为 `pandas.DataFrame` 而不是 `pandas.Series`。
仅当 PyArrow 等于或高于 0.10.0 时，才支持 `BinaryType`。

## <a name="convert-pyspark-dataframes-to-and-from-pandas-dataframes"></a>将 PySpark 数据帧与 Pandas 数据帧相互转换

使用 `toPandas()` 将 PySpark 数据帧转换为 Pandas 数据帧时，以及使用 `createDataFrame(pandas_df)` 从 Pandas 数据帧创建 PySpark 数据帧时，可使用 Arrow 进行优化。
若要将 Arrow 用于这些方法，请将 [Spark 配置](../../../clusters/configure.md#spark-config) `spark.sql.execution.arrow.enabled` 设置为 `true`。
默认情况下，此配置处于禁用状态。

此外，如果在 Spark 内进行计算之前发生错误，则 `spark.sql.execution.arrow.enabled` 启用的优化可能会回退到非 Arrow 实现。
可以使用 Spark 配置 `spark.sql.execution.arrow.fallback.enabled` 来控制此行为。

### <a name="example"></a>示例

```python
import numpy as np
import pandas as pd

# Enable Arrow-based columnar data transfers
spark.conf.set("spark.sql.execution.arrow.enabled", "true")

# Generate a pandas DataFrame
pdf = pd.DataFrame(np.random.rand(100, 3))

# Create a Spark DataFrame from a pandas DataFrame using Arrow
df = spark.createDataFrame(pdf)

# Convert the Spark DataFrame back to a pandas DataFrame using Arrow
result_pdf = df.select("*").toPandas()
```

使用 Arrow 优化生成的结果与未启用 Arrow 时的结果相同。 即使使用 Arrow，`toPandas()` 也会将数据帧中的所有记录收集到驱动程序中，因此应该对数据的一小部分执行此操作。

另外，并非所有 Spark 数据类型都受支持。如果列的类型不受支持，则可能会引发错误。 如果在 `createDataFrame()` 期间发生错误，Spark 将回退到不使用 Arrow 创建数据帧。