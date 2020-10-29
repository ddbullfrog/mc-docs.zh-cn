---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/04/2020
title: SparkR 1.6 概述 - Azure Databricks
ms.openlocfilehash: 10c049cac08c88454b4ecf69e3f76b72e61aae81
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472791"
---
# <a name="sparkr-16-overview"></a>SparkR 1.6 概述

> [!NOTE]
>
> 有关最新 SparkR 库的信息，请参阅[适用于 R 开发人员的 Azure Databricks](../../latest/sparkr/index.md)。

SparkR 是一个 R 包，它提供一个轻型前端，以用于通过 R 使用 Apache Spark。从 Spark 1.5.1 开始，SparkR 提供分布式数据帧实现，该实现支持选择、筛选和聚合等操作（类似于 R 数据帧和 dplyr），但针对的是大型数据集。 SparkR 还使用 MLlib 支持分布式机器学习。

## <a name="creating-sparkr-dataframes"></a>创建 SparkR 数据帧

应用程序可以从本地 R 数据帧或数据源创建数据帧，也可以使用 Spark SQL 查询创建数据帧。

创建数据帧的最简单方法是将本地 R 数据帧转换为 SparkR 数据帧。 具体来说，我们可以使用“创建数据帧”并传入本地 R 数据帧，来创建 SparkR 数据帧。 例如，下面的单元使用 R 中的 faithful 数据集创建一个数据帧。

```r
df <- createDataFrame(sqlContext, faithful)

# Displays the content of the DataFrame to stdout
head(df)
```

### <a name="from-data-sources-using-spark-sql"></a>使用 Spark SQL 从数据源创建

从数据源创建数据帧的常规方法为 read.df。
此方法采用 SQLContext、要加载的文件的路径和数据源类型作为参数。 SparkR 以原生方式支持读取 JSON 和 Parquet 文件，并且你可以通过 Spark 包找到常用文件格式（如 CSV 和 Avro）的数据源连接器。

```bash
%fs rm dbfs:/tmp/people.json
```

```bash
%fs put dbfs:/tmp/people.json
'{"age": 10, "name": "John"}
{"age": 20, "name": "Jane"}
{"age": 30, "name": "Andy"}'
```

```r
people <- read.df(sqlContext, "dbfs:/tmp/people.json", source="json")
```

SparkR 会自动从 JSON 文件推断架构。

```r
printSchema(people)
```

```r
display(people)
```

#### <a name="using-data-source-connectors-with-spark-packages"></a>通过 Spark 包使用数据源连接器创建

例如，我们将使用 Spark CSV 包加载 CSV 文件。 可[在此处找到 Databricks 提供的 Spark 包的列表](https://spark-packages.org/user/databricks)。

```r
diamonds <- read.df(sqlContext, "/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv",
                    source = "com.databricks.spark.csv", header="true", inferSchema = "true")
head(diamonds)
```

数据源 API 也可用于将数据帧保存成多种文件格式。 例如，我们可以使用 write df 将上一个示例中的数据帧保存到 Parquet 文件

```bash
%fs rm -r dbfs:/tmp/people.parquet
```

```r
write.df(people, path="dbfs:/tmp/people.parquet", source="parquet", mode="overwrite")
```

```bash
%fs ls dbfs:/tmp/people.parquet
```

### <a name="from-spark-sql-queries"></a>通过 Spark SQL 查询创建

还可以使用 Spark SQL 查询创建 SparkR 数据帧。

```r
# Register earlier df as temp view
createOrReplaceTempView(people, "peopleTemp")
```

```r
# Create a df consisting of only the 'age' column using a Spark SQL query
age <- sql(sqlContext, "SELECT age FROM peopleTemp")
head(age)
```

```r
# Resulting df is a SparkR df
str(age)
```

## <a name="dataframe-operations"></a>数据帧操作

SparkR 数据帧支持许多函数执行结构化数据处理。 此处提供了一些基本示例，完整的列表可以在 [API 文档](https://spark.apache.org/docs/latest/api/R/)中找到。

### <a name="selecting-rows-and-columns"></a>选择行和列

```r
# Create DataFrame
df <- createDataFrame(sqlContext, faithful)
df
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

SparkR 数据帧支持许多常用函数，以便在分组后聚合数据。 例如，我们可以计算每个等待时间在 faithful 数据集中出现的次数。

```r
head(count(groupBy(df, df$waiting)))
```

```r
# We can also sort the output from the aggregation to get the most common waiting times
waiting_counts <- count(groupBy(df, df$waiting))
head(arrange(waiting_counts, desc(waiting_counts$count)))
```

### <a name="column-operations"></a>列操作

SparkR 提供了许多函数，这些函数可直接应用于列以进行数据处理和聚合。 下面的示例演示基本算术函数的用法。

```r
# Convert waiting time from hours to seconds.
# You can assign this to a new column in the same DataFrame
df$waiting_secs <- df$waiting * 60
head(df)
```

## <a name="machine-learning"></a>机器学习

从 Spark 1.5 起，SparkR 允许使用 glm() 函数在 SparkR 数据帧上拟合通用线性模型。 在内部，SparkR 使用 MLlib 训练指定系列的模型。 对于模型拟合，我们支持可用 R 公式运算符的子集，包括“~”、“.”、“+”和“-”。

在内部，SparkR 会自动对分类特征执行 one-hot 编码，这样便不需要手动执行此操作。
除了 String 和 Double 类型的特征以外，还可以在 MLlib 矢量特征上进行拟合，以便与其他 MLlib 组件兼容。

下面的示例演示如何使用 SparkR 生成高斯 GLM 模型。 若要运行线性回归，请将系列设置为“gaussian”。 若要运行逻辑回归，请将系列设置为“binomial”。

```r
# Create the DataFrame
df <- createDataFrame(sqlContext, iris)

# Fit a linear model over the dataset.
model <- glm(Sepal_Length ~ Sepal_Width + Species, data = df, family = "gaussian")

# Model coefficients are returned in a similar format to R's native glm().
summary(model)
```

## <a name="converting-local-r-data-frames-to-sparkr-dataframes"></a>将本地 R 数据帧转换为 SparkR 数据帧

可以使用 `createDataFrame` 将本地 R 数据帧转换为 SparkR 数据帧。

```r
# Create SparkR DataFrame using localDF
convertedSparkDF <- createDataFrame(sqlContext, localDF)
str(convertedSparkDF)
```

```r
# Another example: Create SparkR DataFrame with a local R data frame
anotherSparkDF <- createDataFrame(sqlContext, data.frame(surname = c("Tukey", "Venables", "Tierney", "Ripley", "McNeil"),
                                                         nationality = c("US", "Australia", "US", "UK", "Australia"),
                                                         deceased = c("yes", rep("no", 4))))
count(anotherSparkDF)
```