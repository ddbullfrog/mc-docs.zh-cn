---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 用户定义的函数 - Scala - Azure Databricks
description: 了解如何实现 Scala 用户定义的函数，以便在 Azure Databricks 中从 Apache Spark SQL 代码使用。
ms.openlocfilehash: 5cd24435e65451aba20400178738cefe151be8e4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472743"
---
# <a name="user-defined-functions---scala"></a>用户定义的函数 - Scala

本文包含 Scala 用户定义的函数 (UDF) 示例。 其中介绍了如何注册 UDF、如何调用 UDF 以及有关 Spark SQL 中子表达式计算顺序的注意事项。

## <a name="register-a-function-as-a-udf"></a>将函数注册为 UDF

```scala
val squared = (s: Long) => {
  s * s
}
spark.udf.register("square", squared)
```

## <a name="call-the-udf-in-spark-sql"></a>在 Spark SQL 中调用 UDF

```scala
spark.range(1, 20).createOrReplaceTempView("test")
```

```sql
%sql select id, square(id) as id_squared from test
```

## <a name="use-udf-with-dataframes"></a>将 UDF 与数据帧配合使用

```scala
import org.apache.spark.sql.functions.{col, udf}
val squared = udf((s: Long) => s * s)
display(spark.range(1, 20).select(squared(col("id")) as "id_squared"))
```

## <a name="evaluation-order-and-null-checking"></a>计算顺序和 NULL 检查

Spark SQL（包括 SQL、数据帧和数据集 API）不保证子表达式的计算顺序。 具体而言，运算符或函数的输入不一定按从左到右或任何其他固定顺序进行计算。 例如，逻辑 `AND` 和 `OR` 表达式没有从左到右的“简化”语义。

因此，依赖于布尔表达式的副作用或计算顺序以及 `WHERE` 和 `HAVING` 子句的顺序是危险的，因为在查询优化和规划过程中，这些表达式和子句可以重新排序。 具体而言，如果 UDF 依赖于 SQL 中的简化语义进行 NULL 检查，则不能保证在调用 UDF 之前执行 NULL 检查。 例如，

```scala
spark.udf.register("strlen", (s: String) => s.length)
spark.sql("select s from test1 where s is not null and strlen(s) > 1") // no guarantee
```

这个 `WHERE` 子句并不保证在筛选出 NULL 后调用 `strlen` UDF。

若要执行正确的 NULL 检查，建议执行以下操作之一：

* 使 UDF 本身能够识别 NULL，并在 UDF 本身的内部进行 NULL 检查
* 使用 `IF` 或 `CASE WHEN` 表达式来执行 NULL 检查并在条件分支中调用 UDF

```scala
spark.udf.register("strlen_nullsafe", (s: String) => if (s != null) s.length else -1)
spark.sql("select s from test1 where s is not null and strlen_nullsafe(s) > 1") // ok
spark.sql("select s from test1 where if(s is not null, strlen(s), null) > 1")   // ok
```