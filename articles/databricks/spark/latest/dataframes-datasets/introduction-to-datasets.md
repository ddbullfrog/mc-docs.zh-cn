---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/04/2020
title: 数据集简介 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Scala 编程语言处理 Apache Spark 数据集 API。
ms.openlocfilehash: f0348eecf5eabdae760e6723f0403987a79c054e
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472779"
---
# <a name="introduction-to-datasets"></a>数据集简介

数据集 API 提供了 RDD 的优点（强类型化、能够使用功能强大的 lambda 函数），以及 Spark SQL 的优化执行引擎的优点。 你可以定义数据集 JVM 对象，然后使用类似于 RDD 的功能转换（`map`、`flatMap`、`filter` 等）对其进行操作。 其优势在于，这些转换现在将应用于 _结构化和强类型化_ 分布式集合，从而允许 Spark 利用 Spark SQL 的执行引擎进行优化，这与 RDD 不同。

## <a name="create-a-dataset"></a>创建数据集

若要将序列转换为数据集，请在序列上调用 `.toDS()`。

```scala
val dataset = Seq(1, 2, 3).toDS()
dataset.show()
```

如果你有一系列 case 类，那么调用 `.toDS()` 将提供一个包含所有必要字段的数据集。

```scala
case class Person(name: String, age: Int)

val personDS = Seq(Person("Max", 33), Person("Adam", 32), Person("Muller", 62)).toDS()
personDS.show()
```

### <a name="create-a-dataset-from-an-rdd"></a>从 RDD 创建数据集

若要将 RDD 转换为数据集，请调用 `rdd.toDS()`。

```scala
val rdd = sc.parallelize(Seq((1, "Spark"), (2, "Databricks")))
val integerDS = rdd.toDS()
integerDS.show()
```

### <a name="create-a-dataset-from-a-dataframe"></a>从数据帧创建数据集

可以调用 `df.as[SomeCaseClass]`，将数据帧转换为数据集。

```scala
case class Company(name: String, foundingYear: Int, numEmployees: Int)
val inputSeq = Seq(Company("ABC", 1998, 310), Company("XYZ", 1983, 904), Company("NOP", 2005, 83))
val df = sc.parallelize(inputSeq).toDF()

val companyDS = df.as[Company]
companyDS.show()
```

也可以在不使用 `case class` 将数据帧转换为数据集时处理元组。

```scala
val rdd = sc.parallelize(Seq((1, "Spark"), (2, "Databricks"), (3, "Notebook")))
val df = rdd.toDF("Id", "Name")

val dataset = df.as[(Int, String)]
dataset.show()
```

## <a name="work-with-datasets"></a>使用数据集

### <a name="word-count-example"></a>字数统计示例

```scala
val wordsDataset = sc.parallelize(Seq("Spark I am your father", "May the spark be with you", "Spark I am your father")).toDS()
val groupedDataset = wordsDataset.flatMap(_.toLowerCase.split(" "))
                                 .filter(_ != "")
                                 .groupBy("value")
val countsDataset = groupedDataset.count()
countsDataset.show()
```

### <a name="join-datasets"></a>联接数据集

下面的示例演示的操作包括：

* 联合多个数据集
* 通过特定列对条件组进行内联
* 对分组的数据集执行自定义聚合（求平均值）。

这些示例仅使用数据集 API 来演示所有可用的操作。 实际上，使用数据帧进行聚合比使用 `mapGroups` 进行自定义聚合更简单快捷。
下一部分将详细介绍如何将数据集转换为数据帧以及如何使用数据帧 API 进行聚合。

```scala
case class Employee(name: String, age: Int, departmentId: Int, salary: Double)
case class Department(id: Int, name: String)

case class Record(name: String, age: Int, salary: Double, departmentId: Int, departmentName: String)
case class ResultSet(departmentId: Int, departmentName: String, avgSalary: Double)

val employeeDataSet1 = sc.parallelize(Seq(Employee("Max", 22, 1, 100000.0), Employee("Adam", 33, 2, 93000.0), Employee("Eve", 35, 2, 89999.0), Employee("Muller", 39, 3, 120000.0))).toDS()
val employeeDataSet2 = sc.parallelize(Seq(Employee("John", 26, 1, 990000.0), Employee("Joe", 38, 3, 115000.0))).toDS()
val departmentDataSet = sc.parallelize(Seq(Department(1, "Engineering"), Department(2, "Marketing"), Department(3, "Sales"))).toDS()

val employeeDataset = employeeDataSet1.union(employeeDataSet2)

def averageSalary(key: (Int, String), iterator: Iterator[Record]): ResultSet = {
  val (total, count) = iterator.foldLeft(0.0, 0.0) {
      case ((total, count), x) => (total + x.salary, count + 1)
  }
  ResultSet(key._1, key._2, total/count)
}

val averageSalaryDataset = employeeDataset.joinWith(departmentDataSet, $"departmentId" === $"id", "inner")
                                          .map(record => Record(record._1.name, record._1.age, record._1.salary, record._1.departmentId, record._2.name))
                                          .filter(record => record.age > 25)
                                          .groupBy($"departmentId", $"departmentName")
                                          .avg()

averageSalaryDataset.show()
```

## <a name="convert-a-dataset-to-a-dataframe"></a>将数据集转换为数据帧

上述 2 个示例演示了如何使用纯数据集 API。 你还可以轻松地从数据集转到数据帧，并利用数据帧 API。 下面的示例展示了同时使用数据集和数据帧 API 的字数统计示例。

```scala
import org.apache.spark.sql.functions._

val wordsDataset = sc.parallelize(Seq("Spark I am your father", "May the spark be with you", "Spark I am your father")).toDS()
val result = wordsDataset
              .flatMap(_.split(" "))               // Split on whitespace
              .filter(_ != "")                     // Filter empty words
              .map(_.toLowerCase())
              .toDF()                              // Convert to DataFrame to perform aggregation / sorting
              .groupBy($"value")                   // Count number of occurrences of each word
              .agg(count("*") as "numOccurances")
              .orderBy($"numOccurances" desc)      // Show most common words first
result.show()
```