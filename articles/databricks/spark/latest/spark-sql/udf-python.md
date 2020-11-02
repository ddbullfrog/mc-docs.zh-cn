---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 用户定义的函数 - Python - Azure Databricks
description: 了解如何实现 Python 用户定义函数，以便在 Azure Databricks 的 Apache Spark SQL 代码中使用。
ms.openlocfilehash: fe4afc667841d66668e36766a5041b0ffd36bbe4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472742"
---
# <a name="user-defined-functions---python"></a>用户定义的函数 - Python

本文包含 Python 用户定义函数 (UDF) 示例。 其中介绍了如何注册 UDF、如何调用 UDF 以及有关 Spark SQL 中子表达式计算顺序的注意事项。

## <a name="register-a-function-as-a-udf"></a>将函数注册为 UDF

```python
def squared(s):
  return s * s
spark.udf.register("squaredWithPython", squared)
```

可以选择设置 UDF 的返回类型。 默认返回类型为 `StringType`。

```python
from pyspark.sql.types import LongType
def squared_typed(s):
  return s * s
spark.udf.register("squaredWithPython", squared_typed, LongType())
```

## <a name="call-the-udf-in-spark-sql"></a>在 Spark SQL 中调用 UDF

```python
spark.range(1, 20).createOrReplaceTempView("test")
```

```sql
%sql select id, squaredWithPython(id) as id_squared from test
```

## <a name="use-udf-with-dataframes"></a>将 UDF 与数据帧配合使用

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import LongType
squared_udf = udf(squared, LongType())
df = spark.table("test")
display(df.select("id", squared_udf("id").alias("id_squared")))
```

另外，还可以使用注释语法声明同一 UDF：

```python
from pyspark.sql.functions import udf
@udf("long")
def squared_udf(s):
  return s * s
df = spark.table("test")
display(df.select("id", squared_udf("id").alias("id_squared")))
```

## <a name="evaluation-order-and-null-checking"></a>计算顺序和 NULL 检查

Spark SQL（包括 SQL、数据帧和数据集 API）不保证子表达式的计算顺序。 具体而言，运算符或函数的输入不一定按从左到右的顺序或任何其他固定顺序进行计算。 例如，逻辑 `AND` 和 `OR` 表达式没有从左到右的“短路”语义。

因此，依赖于布尔表达式计算的副作用或顺序以及 `WHERE` 和 `HAVING` 子句的顺序是危险的，因为在查询优化和规划过程中，这些表达式和子句可能重新排序。 具体而言，如果 UDF 依赖于 SQL 中的短路语义进行 NULL 检查，则不能保证在调用 UDF 之前执行 NULL 检查。 例如，

```python
spark.udf.register("strlen", lambda s: len(s), "int")
spark.sql("select s from test1 where s is not null and strlen(s) > 1") # no guarantee
```

这个 `WHERE` 子句并不保证在筛选掉 NULL 后调用 `strlen` UDF。

若要执行正确的 NULL 检查，建议执行以下操作之一：

* 使 UDF 自身能够识别 NULL，在 UDF 自身内部进行 NULL 检查
* 使用 `IF` 或 `CASE WHEN` 表达式来执行 NULL 检查并在条件分支中调用 UDF

```python
spark.udf.register("strlen_nullsafe", lambda s: len(s) if not s is None else -1, "int")
spark.sql("select s from test1 where s is not null and strlen_nullsafe(s) > 1") // ok
spark.sql("select s from test1 where if(s is not null, strlen(s), null) > 1")   // ok
```