---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: DataFrames 简介 - Scala - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Scala 编程语言处理 Apache Spark DataFrames。
ms.openlocfilehash: ca3fbf8e4a00e056044118594ba30e087e2f3862
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473030"
---
# <a name="introduction-to-dataframes---scala"></a><a id="dataframes-scala"> </a><a id="introduction-to-dataframes---scala"> </a>DataFrames 简介 - Scala

本文演示了使用 Scala 的许多常用 Spark DataFrame 函数。

## <a name="create-dataframes"></a>创建数据帧

```scala
// Create the case classes for our domain
case class Department(id: String, name: String)
case class Employee(firstName: String, lastName: String, email: String, salary: Int)
case class DepartmentWithEmployees(department: Department, employees: Seq[Employee])

// Create the Departments
val department1 = new Department("123456", "Computer Science")
val department2 = new Department("789012", "Mechanical Engineering")
val department3 = new Department("345678", "Theater and Drama")
val department4 = new Department("901234", "Indoor Recreation")

// Create the Employees
val employee1 = new Employee("michael", "armbrust", "no-reply@berkeley.edu", 100000)
val employee2 = new Employee("xiangrui", "meng", "no-reply@stanford.edu", 120000)
val employee3 = new Employee("matei", null, "no-reply@waterloo.edu", 140000)
val employee4 = new Employee(null, "wendell", "no-reply@princeton.edu", 160000)
val employee5 = new Employee("michael", "jackson", "no-reply@neverla.nd", 80000)

// Create the DepartmentWithEmployees instances from Departments and Employees
val departmentWithEmployees1 = new DepartmentWithEmployees(department1, Seq(employee1, employee2))
val departmentWithEmployees2 = new DepartmentWithEmployees(department2, Seq(employee3, employee4))
val departmentWithEmployees3 = new DepartmentWithEmployees(department3, Seq(employee5, employee4))
val departmentWithEmployees4 = new DepartmentWithEmployees(department4, Seq(employee2, employee3))
```

### <a name="create-dataframes-from-a-list-of-the-case-classes"></a>根据案例类列表创建 DataFrames

```scala
val departmentsWithEmployeesSeq1 = Seq(departmentWithEmployees1, departmentWithEmployees2)
val df1 = departmentsWithEmployeesSeq1.toDF()
display(df1)

val departmentsWithEmployeesSeq2 = Seq(departmentWithEmployees3, departmentWithEmployees4)
val df2 = departmentsWithEmployeesSeq2.toDF()
display(df2)
```

## <a name="work-with-dataframes"></a>使用数据帧

### <a name="union-two-dataframes"></a>合并两个数据帧

```scala
val unionDF = df1.union(df2)
display(unionDF)
```

### <a name="write-the-unioned-dataframe-to-a-parquet-file"></a>将合并的数据帧写入 Parquet 文件

```scala
// Remove the file if it exists
dbutils.fs.rm("/tmp/databricks-df-example.parquet", true)
unionDF.write.parquet("/tmp/databricks-df-example.parquet")
```

### <a name="read-a-dataframe-from-the-parquet-file"></a>从 Parquet 文件读取数据帧

```scala
val parquetDF = spark.read.parquet("/tmp/databricks-df-example.parquet")
```

### <a name="explode-the-employees-column"></a>[展开](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$)员工列

```scala
import org.apache.spark.sql.functions._

val explodeDF = parquetDF.select(explode($"employees"))
display(explodeDF)
```

### <a name="flatten-the-fields-of-the-employee-class-into-columns"></a>将员工类的字段合并为列

```scala
val flattenDF = explodeDF.select($"col.*")
flattenDF.show()
```

```
+---------+--------+--------------------+------+
|firstName|lastName|               email|salary|
+---------+--------+--------------------+------+
|    matei|    null|no-reply@waterloo...|140000|
|     null| wendell|no-reply@princeto...|160000|
|  michael|armbrust|no-reply@berkeley...|100000|
| xiangrui|    meng|no-reply@stanford...|120000|
|  michael| jackson| no-reply@neverla.nd| 80000|
|     null| wendell|no-reply@princeto...|160000|
| xiangrui|    meng|no-reply@stanford...|120000|
|    matei|    null|no-reply@waterloo...|140000|
+---------+--------+--------------------+------+
```

### <a name="use-filter-to-return-the-rows-that-match-a-predicate"></a>使用 `filter()` 返回与谓词匹配的行

```scala
val filterDF = flattenDF
  .filter($"firstName" === "xiangrui" || $"firstName" === "michael")
  .sort($"lastName".asc)
display(filterDF)
```

### <a name="the-where-clause-is-equivalent-to-filter"></a>`where()` 子句等效于 `filter()`

```scala
val whereDF = flattenDF
  .where($"firstName" === "xiangrui" || $"firstName" === "michael")
  .sort($"lastName".asc)
display(whereDF)
```

### <a name="replace-null-values-with----using-dataframe-na-function"></a>使用数据帧 Na 函数将 `null` 值替换为 `--`

```scala
val nonNullDF = flattenDF.na.fill("--")
display(nonNullDF)
```

### <a name="retrieve-rows-with-missing-firstname-or-lastname"></a>检索缺少名字或姓氏的行

```scala
val filterNonNullDF = nonNullDF.filter($"firstName" === "--" || $"lastName" === "--").sort($"email".asc)
display(filterNonNullDF)
```

### <a name="example-aggregations-using-agg-and-countdistinct"></a>使用 `agg()` 和  的示例聚合`countDistinct()`

```scala
// Find the distinct last names for each first name
val countDistinctDF = nonNullDF.select($"firstName", $"lastName")
  .groupBy($"firstName")
  .agg(countDistinct($"lastName") as "distinct_last_names")
display(countDistinctDF)
```

### <a name="compare-the-dataframe-and-sql-query-physical-plans"></a>比较数据帧和 SQL 查询的物理计划

> [!TIP]
>
> 它们应相同。

```scala
countDistinctDF.explain()
```

```scala
// register the DataFrame as a temp view so that we can query it using SQL
nonNullDF.createOrReplaceTempView("databricks_df_example")

spark.sql("""
  SELECT firstName, count(distinct lastName) as distinct_last_names
  FROM databricks_df_example
  GROUP BY firstName
""").explain
```

### <a name="sum-up-all-the-salaries"></a>计算所有薪水的总和

```scala
val salarySumDF = nonNullDF.agg("salary" -> "sum")
display(salarySumDF)
```

### <a name="print-the-summary-statistics-for-the-salaries"></a>输出薪水的汇总统计信息

```scala
nonNullDF.describe("salary").show()
```

### <a name="cleanup-remove-the-parquet-file"></a>清理：删除 Parquet 文件

```scala
dbutils.fs.rm("/tmp/databricks-df-example.parquet", true)
```

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

本常见问题解答使用可用的 API 解决了常见的用例和示例用法。 有关 API 的详细说明，请参阅 [DataFrameReader](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameReader) 和 [DataFrameWriter](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameWriter) 文档。

**如何使用 DataFrame UDF 获得更好的性能？**

如果在可用的内置函数中存在这些功能，使用它们会有更好的表现。

我们使用内置函数和 `withColumn()` API 添加新列。
我们还可以在转换后使用 `withColumnRenamed()` 替换现有的列。

```scala
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types._
import org.apache.spark.sql._
import org.apache.hadoop.io.LongWritable
import org.apache.hadoop.io.Text
import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat

// Build an example DataFrame dataset to work with.
dbutils.fs.rm("/tmp/dataframe_sample.csv", true)
dbutils.fs.put("/tmp/dataframe_sample.csv", """
id|end_date|start_date|location
1|2015-10-14 00:00:00|2015-09-14 00:00:00|CA-SF
2|2015-10-15 01:00:20|2015-08-14 00:00:00|CA-SD
3|2015-10-16 02:30:00|2015-01-14 00:00:00|NY-NY
4|2015-10-17 03:00:20|2015-02-14 00:00:00|NY-NY
5|2015-10-18 04:30:00|2014-04-14 00:00:00|CA-LA
""", true)

val conf = new Configuration
conf.set("textinputformat.record.delimiter", "\n")
val rdd = sc.newAPIHadoopFile("/tmp/dataframe_sample.csv", classOf[TextInputFormat], classOf[LongWritable], classOf[Text], conf).map(_._2.toString).filter(_.nonEmpty)

val header = rdd.first()
// Parse the header line
val rdd_noheader = rdd.filter(x => !x.contains("id"))
// Convert the RDD[String] to an RDD[Rows]. Create an array using the delimiter and use Row.fromSeq()
val row_rdd = rdd_noheader.map(x => x.split('|')).map(x => Row.fromSeq(x))

val df_schema =
  StructType(
    header.split('|').map(fieldName => StructField(fieldName, StringType, true)))

var df = spark.createDataFrame(row_rdd, df_schema)
df.printSchema
```

```scala
// Instead of registering a UDF, call the builtin functions to perform operations on the columns.
// This will provide a performance improvement as the builtins compile and run in the platform's JVM.

// Convert to a Date type
val timestamp2datetype: (Column) => Column = (x) => { to_date(x) }
df = df.withColumn("date", timestamp2datetype(col("end_date")))

// Parse out the date only
val timestamp2date: (Column) => Column = (x) => { regexp_replace(x," (\\d+)[:](\\d+)[:](\\d+).*$", "") }
df = df.withColumn("date_only", timestamp2date(col("end_date")))

// Split a string and index a field
val parse_city: (Column) => Column = (x) => { split(x, "-")(1) }
df = df.withColumn("city", parse_city(col("location")))

// Perform a date diff function
val dateDiff: (Column, Column) => Column = (x, y) => { datediff(to_date(y), to_date(x)) }
df = df.withColumn("date_diff", dateDiff(col("start_date"), col("end_date")))
```

```scala
df.createOrReplaceTempView("sample_df")
display(sql("select * from sample_df"))
```

**我想将 DataFrame 转换回 JSON 字符串以发送回 Kafka。**

有一个 `toJSON()` 函数，该函数使用列名和架构返回 JSON 字符串的 RDD，以生成 JSON 记录。

```scala
val rdd_json = df.toJSON
rdd_json.take(2).foreach(println)
```

**我的 UDF 需要一个参数，包括要操作的列。如何传递这个参数？**

有一个名为 `lit()` 的函数可以创建一个静态列。

```scala
val add_n = udf((x: Integer, y: Integer) => x + y)

// We register a UDF that adds a column to the DataFrame, and we cast the id column to an Integer type.
df = df.withColumn("id_offset", add_n(lit(1000), col("id").cast("int")))
display(df)
```

```scala
val last_n_days = udf((x: Integer, y: Integer) => {
  if (x < y) true else false
})

//last_n_days = udf(lambda x, y: True if x < y else False, BooleanType())

val df_filtered = df.filter(last_n_days(col("date_diff"), lit(90)))
display(df_filtered)
```

**我在 Hive 元存储中有一个表，我想以 DataFrame 的形式访问这个表。定义此 DataFrame 的最佳方式是什么？**

有多种方法可以从注册的表定义 DataFrame。
调用 `table(tableName)` 或使用 SQL 查询选择和筛选特定的列：

```scala
// Both return DataFrame types
val df_1 = table("sample_df")
val df_2 = spark.sql("select * from sample_df")
```

**我想清除当前群集中的所有缓存表。**

有一个 API 可以在全局级别或每个表级别执行此操作。

```scala
sqlContext.clearCache()
sqlContext.cacheTable("sample_df")
sqlContext.uncacheTable("sample_df")
```

**我想计算列的聚合。最佳方式是什么？**

有一个名为 `agg(*exprs)` 的 API，它需要一个列名和表达式的列表，用于你想要计算的聚合类型。 你可以利用上面提到的内置函数作为每个列的表达式的一部分。

```scala
// Provide the min, count, and avg and groupBy the location column. Diplay the results
var agg_df = df.groupBy("location").agg(min("id"), count("id"), avg("date_diff"))
display(agg_df)
```

**我想把 DataFrames 写出到 Parquet，但想在某一列上进行分区。**

可以使用以下 API 来实现此目的。 确保代码不会使用数据集创建大量的分区列，否则元数据的开销会造成明显的减速。 如果此目录后面有一个 SQL 表，则需要在查询前调用 `refresh table <table-name>` 来更新元数据。

```scala
df = df.withColumn("end_month", month(col("end_date")))
df = df.withColumn("end_year", year(col("end_date")))
dbutils.fs.rm("/tmp/sample_table", true)
df.write.partitionBy("end_year", "end_month").parquet("/tmp/sample_table")
display(dbutils.fs.ls("/tmp/sample_table"))
```

**如何正确处理要筛选出 NULL 数据的情况？**

你可以使用 `filter()` 并提供类似于 SQL 查询的语法。

```scala
val null_item_schema = StructType(Array(StructField("col1", StringType, true),
                               StructField("col2", IntegerType, true)))

val null_dataset = sc.parallelize(Array(("test", 1 ), (null, 2))).map(x => Row.fromTuple(x))
val null_df = spark.createDataFrame(null_dataset, null_item_schema)
display(null_df.filter("col1 IS NOT NULL"))
```

**如何使用 `csv` 或 `spark-avro` 库来推断架构？**

有一个 `inferSchema` 选项标志。 提供标头可让你适当地命名列。

```scala
val adult_df = spark.read.
    format("csv").
    option("header", "false").
    option("inferSchema", "true").load("dbfs:/databricks-datasets/adult/adult.data")
adult_df.printSchema()
```

**你有一个以符号分隔的字符串数据集想要转换为它们的数据类型。你将如何实现这一目标？**

使用 RDD API 筛选出格式错误的行，并将值映射到适当的类型。