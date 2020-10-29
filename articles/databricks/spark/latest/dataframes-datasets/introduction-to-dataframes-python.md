---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 数据帧简介 - Python - Azure Databricks
description: 了解如何在 Azure Databricks 中通过 Python 使用 Apache Spark 数据帧。
ms.openlocfilehash: cba0b1302a064e813cbcf0e4d5633450bd08d1f3
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472782"
---
# <a name="introduction-to-dataframes---python"></a><a id="dataframes-python"> </a><a id="introduction-to-dataframes---python"> </a>数据帧简介 - Python

本文演示了许多使用 Python 的常见 Spark 数据帧函数。

## <a name="create-dataframes"></a>创建数据帧

```python
# import pyspark class Row from module sql
from pyspark.sql import *

# Create Example Data - Departments and Employees

# Create the Departments
department1 = Row(id='123456', name='Computer Science')
department2 = Row(id='789012', name='Mechanical Engineering')
department3 = Row(id='345678', name='Theater and Drama')
department4 = Row(id='901234', name='Indoor Recreation')

# Create the Employees
Employee = Row("firstName", "lastName", "email", "salary")
employee1 = Employee('michael', 'armbrust', 'no-reply@berkeley.edu', 100000)
employee2 = Employee('xiangrui', 'meng', 'no-reply@stanford.edu', 120000)
employee3 = Employee('matei', None, 'no-reply@waterloo.edu', 140000)
employee4 = Employee(None, 'wendell', 'no-reply@berkeley.edu', 160000)
employee5 = Employee('michael', 'jackson', 'no-reply@neverla.nd', 80000)

# Create the DepartmentWithEmployees instances from Departments and Employees
departmentWithEmployees1 = Row(department=department1, employees=[employee1, employee2])
departmentWithEmployees2 = Row(department=department2, employees=[employee3, employee4])
departmentWithEmployees3 = Row(department=department3, employees=[employee5, employee4])
departmentWithEmployees4 = Row(department=department4, employees=[employee2, employee3])

print(department1)
print(employee2)
print(departmentWithEmployees1.employees[0].email)
```

### <a name="create-dataframes-from-a-list-of-the-rows"></a>从行列表创建数据帧

```python
departmentsWithEmployeesSeq1 = [departmentWithEmployees1, departmentWithEmployees2]
df1 = spark.createDataFrame(departmentsWithEmployeesSeq1)

display(df1)

departmentsWithEmployeesSeq2 = [departmentWithEmployees3, departmentWithEmployees4]
df2 = spark.createDataFrame(departmentsWithEmployeesSeq2)

display(df2)
```

## <a name="work-with-dataframes"></a>使用数据帧

### <a name="union-two-dataframes"></a>合并两个数据帧

```python
unionDF = df1.union(df2)
display(unionDF)
```

### <a name="write-the-unioned-dataframe-to-a-parquet-file"></a>将合并的数据帧写入 Parquet 文件

```python
# Remove the file if it exists
dbutils.fs.rm("/tmp/databricks-df-example.parquet", True)
unionDF.write.parquet("/tmp/databricks-df-example.parquet")
```

### <a name="read-a-dataframe-from-the-parquet-file"></a>从 Parquet 文件读取数据帧

```python
parquetDF = spark.read.parquet("/tmp/databricks-df-example.parquet")
display(parquetDF)
```

### <a name="explode-the-employees-column"></a>[分解](https://spark.apache.org/docs/latest/api/python/_modules/pyspark/sql/functions.html#explode) employees 列

```python
from pyspark.sql.functions import explode

explodeDF = unionDF.select(explode("employees").alias("e"))
flattenDF = explodeDF.selectExpr("e.firstName", "e.lastName", "e.email", "e.salary")

flattenDF.show()
```

```
+---------+--------+--------------------+------+
|firstName|lastName|               email|salary|
+---------+--------+--------------------+------+
|  michael|armbrust|no-reply@berkeley...|100000|
| xiangrui|    meng|no-reply@stanford...|120000|
|    matei|    null|no-reply@waterloo...|140000|
|     null| wendell|no-reply@berkeley...|160000|
|  michael| jackson| no-reply@neverla.nd| 80000|
|     null| wendell|no-reply@berkeley...|160000|
| xiangrui|    meng|no-reply@stanford...|120000|
|    matei|    null|no-reply@waterloo...|140000|
+---------+--------+--------------------+------+
```

### <a name="use-filter-to-return-the-rows-that-match-a-predicate"></a>使用 `filter()` 返回与谓词匹配的行

```python
filterDF = flattenDF.filter(flattenDF.firstName == "xiangrui").sort(flattenDF.lastName)
display(filterDF)
```

```python
from pyspark.sql.functions import col, asc

# Use `|` instead of `or`
filterDF = flattenDF.filter((col("firstName") == "xiangrui") | (col("firstName") == "michael")).sort(asc("lastName"))
display(filterDF)
```

### <a name="the-where-clause-is-equivalent-to-filter"></a>`where()` 子句等效于 `filter()`

```python
whereDF = flattenDF.where((col("firstName") == "xiangrui") | (col("firstName") == "michael")).sort(asc("lastName"))
display(whereDF)
```

### <a name="replace-null-values-with----using-dataframe-na-function"></a>使用数据帧 Na 函数将 `null` 值替换为 `--`

```python
nonNullDF = flattenDF.fillna("--")
display(nonNullDF)
```

### <a name="retrieve-only-rows-with-missing-firstname-or-lastname"></a>仅检索缺失 `firstName` 或  的行`lastName`

```python
filterNonNullDF = flattenDF.filter(col("firstName").isNull() | col("lastName").isNull()).sort("email")
display(filterNonNullDF)
```

### <a name="example-aggregations-using-agg-and-countdistinct"></a>使用 `agg()` 和  的示例聚合`countDistinct()`

```python
from pyspark.sql.functions import countDistinct

countDistinctDF = nonNullDF.select("firstName", "lastName")\
  .groupBy("firstName")\
  .agg(countDistinct("lastName").alias("distinct_last_names"))

display(countDistinctDF)
```

### <a name="compare-the-dataframe-and-sql-query-physical-plans"></a>比较数据帧和 SQL 查询的物理计划

> [!TIP]
>
> 它们应相同。

```python
countDistinctDF.explain()
```

```python
# register the DataFrame as a temp view so that we can query it using SQL
nonNullDF.createOrReplaceTempView("databricks_df_example")

# Perform the same query as the DataFrame above and return ``explain``
countDistinctDF_sql = spark.sql('''
  SELECT firstName, count(distinct lastName) AS distinct_last_names
  FROM databricks_df_example
  GROUP BY firstName
''')

countDistinctDF_sql.explain()
```

### <a name="sum-up-all-the-salaries"></a>计算所有薪水的总和

```python
salarySumDF = nonNullDF.agg({"salary" : "sum"})
display(salarySumDF)
```

```python
type(nonNullDF.salary)
```

### <a name="print-the-summary-statistics-for-the-salaries"></a>输出薪水的汇总统计信息

```python
nonNullDF.describe("salary").show()
```

### <a name="an-example-using-pandas-and-matplotlib-integration"></a>一个使用 pandas 和 Matplotlib 集成的示例

```python
import pandas as pd
import matplotlib.pyplot as plt
plt.clf()
pdDF = nonNullDF.toPandas()
pdDF.plot(x='firstName', y='salary', kind='bar', rot=45)
display()
```

### <a name="cleanup-remove-the-parquet-file"></a>清理：删除 Parquet 文件

```python
dbutils.fs.rm("/tmp/databricks-df-example.parquet", True)
```

## <a name="dataframe-faqs"></a>数据帧常见问题解答

本常见问题解答提供了使用可用 API 的常见用例和示例用法。 有关更详细的 API 说明，请参阅 [PySpark 文档](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame)。

**如何使用数据帧 UDF 提高性能？**

如果所需的功能存在于可用的内置函数中，使用这些会提高性能。 用法示例如下所示。 另请参阅 [pyspark.sql.function 文档](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#module-pyspark.sql.functions)。 我们使用内置函数和 `withColumn()` API 添加新列。 我们还可以在转换后使用 `withColumnRenamed()` 替换现有的列。

```python
from pyspark.sql import functions as F
from pyspark.sql.types import *

# Build an example DataFrame dataset to work with.
dbutils.fs.rm("/tmp/dataframe_sample.csv", True)
dbutils.fs.put("/tmp/dataframe_sample.csv", """id|end_date|start_date|location
1|2015-10-14 00:00:00|2015-09-14 00:00:00|CA-SF
2|2015-10-15 01:00:20|2015-08-14 00:00:00|CA-SD
3|2015-10-16 02:30:00|2015-01-14 00:00:00|NY-NY
4|2015-10-17 03:00:20|2015-02-14 00:00:00|NY-NY
5|2015-10-18 04:30:00|2014-04-14 00:00:00|CA-SD
""", True)

df = spark.read.format("csv").options(header='true', delimiter = '|').load("/tmp/dataframe_sample.csv")
df.printSchema()
```

```python
# Instead of registering a UDF, call the builtin functions to perform operations on the columns.
# This will provide a performance improvement as the builtins compile and run in the platform's JVM.

# Convert to a Date type
df = df.withColumn('date', F.to_date(df.end_date))

# Parse out the date only
df = df.withColumn('date_only', F.regexp_replace(df.end_date,' (\d+)[:](\d+)[:](\d+).*$', ''))

# Split a string and index a field
df = df.withColumn('city', F.split(df.location, '-')[1])

# Perform a date diff function
df = df.withColumn('date_diff', F.datediff(F.to_date(df.end_date), F.to_date(df.start_date)))
```

```python
df.createOrReplaceTempView("sample_df")
display(sql("select * from sample_df"))
```

**我想将数据帧转换回 JSON 字符串以发送回 Kafka。**

有一个基础函数 `toJSON()`，该函数使用列名和架构生成 JSON 记录，以返回 JSON 字符串的 RDD。

```python
rdd_json = df.toJSON()
rdd_json.take(2)
```

**我的 UDF 需要一个参数，包括要操作的列。如何传递此参数？**

有一个名为 `lit()` 的函数可用于创建一个常数列。

```python
from pyspark.sql import functions as F

add_n = udf(lambda x, y: x + y, IntegerType())

# We register a UDF that adds a column to the DataFrame, and we cast the id column to an Integer type.
df = df.withColumn('id_offset', add_n(F.lit(1000), df.id.cast(IntegerType())))
```

```python
display(df)
```

```python
# any constants used by UDF will automatically pass through to workers
N = 90
last_n_days = udf(lambda x: x < N, BooleanType())

df_filtered = df.filter(last_n_days(df.date_diff))
display(df_filtered)
```

**我在 Hive 元存储中有一个表，我想以数据帧的形式访问该表。定义此数据帧的最佳方式是什么？**

有多种方法可以从注册的表中定义数据帧。 调用 `table(tableName)` 或使用 SQL 查询选择和筛选特定列：

```python
# Both return DataFrame types
df_1 = table("sample_df")
df_2 = spark.sql("select * from sample_df")
```

**我想清除当前群集中的所有缓存表。**

有一个 API 可用于在全局级别或按表执行此操作。

```python
sqlContext.clearCache()
sqlContext.cacheTable("sample_df")
sqlContext.uncacheTable("sample_df")
```

**我想针对列计算聚合。最佳方式是什么？**

有一个名为 `agg(*exprs)` 的 API，它采用一个列表，其中包含适用于你想要计算的聚合类型的列名和表达式。 文档已在 [pyspark.sql 模块](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html)中提供。 你可以利用上面提到的内置函数作为每个列的表达式的一部分。

```python
# Provide the min, count, and avg and groupBy the location column. Diplay the results
agg_df = df.groupBy("location").agg(F.min("id"), F.count("id"), F.avg("date_diff"))
display(agg_df)
```

**我想将数据帧写出到 Parquet，但想在特定列上进行分区。**

可以使用以下 API 来实现此目的。 确保代码不会使用数据集创建大量的分区列，否则元数据的开销可能会造成明显的减速。 如果有一个 SQL 表基于此目录，则需要在查询前调用 `refresh table <table-name>` 来更新元数据。

```python
df = df.withColumn('end_month', F.month('end_date'))
df = df.withColumn('end_year', F.year('end_date'))
df.write.partitionBy("end_year", "end_month").parquet("/tmp/sample_table")
display(dbutils.fs.ls("/tmp/sample_table"))
```

**如何正确处理要筛选出 NULL 数据的情况？**

可以使用 `filter()` 并提供类似于 SQL 查询的语法。

```python
null_item_schema = StructType([StructField("col1", StringType(), True),
                               StructField("col2", IntegerType(), True)])
null_df = spark.createDataFrame([("test", 1), (None, 2)], null_item_schema)
display(null_df.filter("col1 IS NOT NULL"))
```

**如何使用 `CSV` 或 `spark-avro` 库来推断架构？**

有一个 `inferSchema` 选项标志。 提供标题可确保进行相应的列命名。

```python
adult_df = spark.read.\
    format("com.spark.csv").\
    option("header", "false").\
    option("inferSchema", "true").load("dbfs:/databricks-datasets/adult/adult.data")
adult_df.printSchema()
```

**你有一个带分隔符的字符串数据集，并且你想要将该数据集转换为其数据类型。你将如何实现这一目标？**

使用 RDD API 筛选出格式错误的行，并将值映射到相应的类型。 我们定义了一个函数，该函数使用正则表达式来筛选项。