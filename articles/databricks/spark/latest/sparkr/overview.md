---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: SparkR 概述 - Azure Databricks
description: 了解如何使用 Azure Databricks 中的 SparkR 来处理 R 中的 Apache Spark。
ms.openlocfilehash: b385ed2124651cd9806b8d8bce35f97b87dd389a
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472740"
---
# <a name="sparkr-overview"></a>SparkR 概述

SparkR 是一个 R 包，它提供轻型前端来使用 R 中的 Apache Spark。SparkR 还支持使用 MLlib 的分布式机器学习。

## <a name="sparkr-in-notebooks"></a>笔记本中的 SparkR

* 对于 Spark 2.0 及更高版本，无需将 `sqlContext` 对象显式传递给每个函数调用。 本文使用新语法。 有关旧语法示例，请参阅 [SparkR 1.6 概述](../../1.6/sparkr/overview.md)。
* 对于 Spark 2.2 及更高版本，默认情况下，笔记本不再导入 SparkR，因为 SparkR 函数与其他常用包中名称类似的函数冲突。 若要使用 SparkR，你可以在笔记本中调用 `library(SparkR)`。 已配置 SparkR 会话，并且所有 SparkR 函数都将使用现有会话与连接的群集通信。

## <a name="sparkr-in-spark-submit-jobs"></a>spark-submit 作业中的 SparkR

你可以运行在 Azure Databricks 上使用 SparkR 作为 spark-submit 作业的脚本，只需进行少量代码修改。 有关示例，请参阅 [创建并运行适用于 R 脚本的 spark-submit 作业](../../../dev-tools/api/latest/examples.md#spark-submit-api-example-r)。

## <a name="create-sparkr-dataframes"></a>创建 SparkR 数据帧

可以从本地 R `data.frame`、数据源或使用 Spark SQL 查询创建数据帧。

### <a name="from-a-local-r-dataframe"></a>从本地 R `data.frame` 创建数据帧

创建数据帧最简单的方法是将本地 R `data.frame` 转换为 `SparkDataFrame`。 具体来说，我们可以使用 `createDataFrame`，并传入本地 R `data.frame` 以创建 `SparkDataFrame`。 与大多数其他 SparkR 函数一样，Spark 2.0 中的 `createDataFrame` 语法有所变化。 可以在以下代码片段中查看此语法示例。
有关更多示例，请参阅 [createDataFrame](https://spark.apache.org/docs/latest/api/R/createDataFrame.html)。

```r
library(SparkR)
df <- createDataFrame(faithful)

# Displays the content of the DataFrame to stdout
head(df)
```

### <a name="using-the-data-source-api"></a>使用数据源 API 创建数据帧

从数据源创建数据帧的常规方法是 `read.df`。
此方法需要获取要加载的文件的路径和数据源的类型。
SparkR 支持原生读取 CSV、JSON、文本和 Parquet 文件。

```r
library(SparkR)
diamondsDF <- read.df("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv", source = "csv", header="true", inferSchema = "true")
head(diamondsDF)
```

SparkR 会自动从 CSV 文件推断架构。

#### <a name="adding-a-data-source-connector-with-spark-packages"></a>使用 Spark 包添加数据源连接器

通过 Spark 包，可以找到常用文件格式（如 Avro）的数据源连接器。 例如，使用 [spark-avro 包](https://spark-packages.org/package/databricks/spark-avro)加载 [Avro](https://avro.apache.org/) 文件。 spark-avro 包的可用性取决于群集的[映像版本](../../../runtime/index.md#dbr-overview)。 请参阅 [Avro 文件](../../../data/data-sources/read-avro.md)。

首先获取现有 `data.frame`，将其转换为 Spark 数据帧，并将其另存为 Avro 文件。

```r
require(SparkR)
irisDF <- createDataFrame(iris)
write.df(irisDF, source = "com.databricks.spark.avro", path = "dbfs:/tmp/iris.avro", mode = "overwrite")
```

验证 Avro 文件是否已保存：

```r
%fs ls /tmp/iris
```

现在再次使用 spark-avro 包读回数据。

```r
irisDF2 <- read.df(path = "/tmp/iris.avro", source = "com.databricks.spark.avro")
head(irisDF2)
```

数据源 API 还可用于将数据帧保存为多种文件格式。 例如，可以使用 `write.df` 将上一个示例中的数据帧保存到 Parquet 文件。

```r
write.df(irisDF2, path="dbfs:/tmp/iris.parquet", source="parquet", mode="overwrite")
```

```bash
%fs ls dbfs:/tmp/people.parquet
```

### <a name="from-a-spark-sql-query"></a>从 Spark SQL 查询创建数据帧

还可以使用 Spark SQL 查询创建 SparkR 数据帧。

```r
# Register earlier df as temp view
createOrReplaceTempView(people, "peopleTemp")
```

```r
# Create a df consisting of only the 'age' column using a Spark SQL query
age <- sql("SELECT age FROM peopleTemp")
```

`age` 是 SparkDataFrame。

## <a name="dataframe-operations"></a>数据帧操作

Spark 数据帧支持多种函数来进行结构化数据处理。 下面是一些基本示例。 你可以在 [API 文档](https://spark.apache.org/docs/latest/api/R/)中找到完整列表。

### <a name="select-rows-and-columns"></a>选择行和列

```r
# Import SparkR package if this is a new notebook
require(SparkR)

# Create DataFrame
df <- createDataFrame(faithful)
```

```r
# Select only the "eruptions" column
head(select(df, df$eruptions))
```

```r
# You can also pass in column name as strings
head(select(df, "eruptions"))
```

```r
# Filter the DataFrame to only retain rows with wait times shorter than 50 mins
head(filter(df, df$waiting < 50))
```

### <a name="grouping-and-aggregation"></a>分组和聚合

Spark 数据帧支持多种常用函数，用于在分组后聚合数据。 例如，你可以计算每个等待时间在保真数据集中出现的次数。

```r
head(count(groupBy(df, df$waiting)))
```

```r
# You can also sort the output from the aggregation to get the most common waiting times
waiting_counts <- count(groupBy(df, df$waiting))
head(arrange(waiting_counts, desc(waiting_counts$count)))
```

### <a name="column-operations"></a>列操作

SparkR 提供了许多函数，这些函数可直接应用于列以进行数据处理和聚合。 以下示例演示基本算术函数的用法。

```r
# Convert waiting time from hours to seconds.
# You can assign this to a new column in the same DataFrame
df$waiting_secs <- df$waiting * 60
head(df)
```

## <a name="machine-learning"></a><a id="machine-learning"> </a><a id="r-ml"> </a>机器学习

SparkR 公开大多数 MLLib 算法。 实际上，SparkR 使用 MLlib 来训练模型。

以下示例演示如何使用 SparkR 来生成 gaussian GLM 模型。 若要运行线性回归，请将系列设置为 `"gaussian"`。 若要运行逻辑回归，请将系列设置为 `"binomial"`。 当使用 SparkML GLM 时，SparkR 会自动对分类特征进行独热编码，因此不需要手动执行此操作。
除了 String 和 Double 类型特征以外，还可以针对 MLlib 矢量特征进行拟合，以与其他 MLlib 组件兼容。

```r
# Create the DataFrame
df <- createDataFrame(iris)

# Fit a linear model over the dataset.
model <- glm(Sepal_Length ~ Sepal_Width + Species, data = df, family = "gaussian")

# Model coefficients are returned in a similar format to R's native glm().
summary(model)
```

有关教程，请参阅 [SparkR ML 教程](tutorials/index.md)。