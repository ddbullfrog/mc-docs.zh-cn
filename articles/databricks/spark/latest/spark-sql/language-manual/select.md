---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: Select - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark 和 Delta Lake SQL 语言的 SELECT 语法。
ms.openlocfilehash: 39af7e31ee89f73b72664dd76f233a99dd2bfe99
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472843"
---
# <a name="select"></a>Select

```sql
SELECT [hints, ...] [ALL|DISTINCT] named_expression[, named_expression, ...]
  FROM relation[, relation, ...]
  [lateral_view[, lateral_view, ...]]
  [WHERE boolean_expression]
  [aggregation [HAVING boolean_expression]]
  [ORDER BY sort_expressions]
  [CLUSTER BY expressions]
  [DISTRIBUTE BY expressions]
  [SORT BY sort_expressions]
  [WINDOW named_window[, WINDOW named_window, ...]]
  [LIMIT num_rows]

named_expression:
  : expression [AS alias]

relation:
  | join_relation
  | (table_name|query|relation) [sample] [AS alias]
  : VALUES (expressions)[, (expressions), ...]
        [AS (column_name[, column_name, ...])]

expressions:
  : expression[, expression, ...]

sort_expressions:
  : expression [ASC|DESC][, expression [ASC|DESC], ...]
```

从一个或多个关系输出数据。

关系指的是输入数据的任何来源。 它可以是现有表（或视图）的内容、两个现有表的联接结果，或者是子查询（另一 `SELECT` 语句的结果）。

（Azure Databricks 上的 Delta Lake）在 Delta Lake 中，可以通过指定 `delta.<path-to-table>` 或 `table_name` 来指定关系。 此外，还可以使用 `TIMESTAMP AS OF`、`VERSION AS OF` 或 `@` 语法在表标识符后指定“按时间顺序查看”版本。 有关详细信息，请参阅[查询表的旧快照（按时间顺序查看）](../../../../delta/delta-batch.md#deltatimetravel)。

**`ALL`**

选择关系中的所有匹配行。 默认情况下启用。

**`DISTINCT`**

选择关系中的所有匹配行，然后删除重复的结果。

**`WHERE`**

按谓词筛选行。

**`HAVING`**

按谓词筛选分组后的结果。

**`ORDER BY`**

对一组表达式施加总计排序。 默认排序方向为升序。 不能将此项与 `SORT BY`、`CLUSTER BY` 或 `DISTRIBUTE BY` 一起使用。

**`DISTRIBUTE BY`**

基于一组表达式对关系中的行重新分区。 具有相同表达式值的行将哈希化到同一个工作器。 不能将此项与 `ORDER BY` 或 `CLUSTER BY` 一起使用。

**`SORT BY`**

对每个分区中的一组表达式施加排序。 默认排序方向为升序。 不能将此项与 `ORDER BY` 或 `CLUSTER BY` 一起使用。

**`CLUSTER BY`**

基于一组表达式对关系中的行重新分区，并根据表达式以升序对行排序。 换句话说，这是 `DISTRIBUTE BY` 和 `SORT BY` 的简写，其中所有表达式都按升序排序。 不能将此项与 `ORDER BY`、`DISTRIBUTE BY` 或 `SORT BY` 一起使用。

**`WINDOW`**

将标识符分配给窗口规范。 请参阅[窗口函数](#window-functions)。

**`LIMIT`**

限制返回的行数。

**`VALUES`**

显式指定值，而不是从关系中读取它们。

## <a name="examples"></a>示例

```sql
SELECT * FROM boxes
SELECT width, length FROM boxes WHERE height=3
SELECT DISTINCT width, length FROM boxes WHERE height=3 LIMIT 2
SELECT * FROM VALUES (1, 2, 3) AS (width, length, height)
SELECT * FROM VALUES (1, 2, 3), (2, 3, 4) AS (width, length, height)
SELECT * FROM boxes ORDER BY width
SELECT * FROM boxes DISTRIBUTE BY width SORT BY width
SELECT * FROM boxes CLUSTER BY length
```

## <a name="sampling"></a>采样

```sql
sample:
    | TABLESAMPLE ([integer_expression | decimal_expression] PERCENT)
    : TABLESAMPLE (integer_expression ROWS)
```

对输入数据采样。 这可以表示为百分比（必须介于 0 到 100 之间）或固定数量的输入行。

### <a name="examples"></a>示例

```sql
SELECT * FROM boxes TABLESAMPLE (3 ROWS)
SELECT * FROM boxes TABLESAMPLE (25 PERCENT)
```

## <a name="joins"></a>联接

```sql
join_relation:
    | relation join_type JOIN relation [ON boolean_expression | USING (column_name, column_name) ]
    : relation NATURAL join_type JOIN relation
join_type:
    | INNER
    | [LEFT | RIGHT] SEMI
    | [LEFT | RIGHT | FULL] [OUTER]
    : [LEFT] ANTI
```

**`INNER JOIN`**

从两个关系中选择存在匹配项的所有行。

**`OUTER JOIN`**

从两个关系中选择所有行，并在没有匹配项的那一侧填充 NULL 值。

**`SEMI JOIN`**

仅从存在匹配项的 `SEMI JOIN` 侧选择行。 如果一个行与多个行匹配，则仅返回第一个匹配项。

**`LEFT ANTI JOIN`**

从左侧选择在右侧没有匹配行的行。

### <a name="examples"></a>示例

```sql
SELECT * FROM boxes INNER JOIN rectangles ON boxes.width = rectangles.width
SELECT * FROM boxes FULL OUTER JOIN rectangles USING (width, length)
SELECT * FROM boxes NATURAL JOIN rectangles
```

## <a name="lateral-view"></a>Lateral view

```sql
lateral_view:
    : LATERAL VIEW [OUTER] function_name (expressions)
          table_name [AS (column_name[, column_name, ...])]
```

使用表生成函数为每个输入行生成零个或多个输出行。 与 `LATERAL VIEW` 一起使用的最常见的内置函数是 `explode`。

**`LATERAL VIEW OUTER`**

即使函数返回零行，也会生成一个包含 NULL 值的行。

### <a name="examples"></a>示例

```sql
SELECT * FROM boxes LATERAL VIEW explode(Array(1, 2, 3)) my_view
SELECT name, my_view.grade FROM students LATERAL VIEW OUTER explode(grades) my_view AS grade
```

## <a name="aggregation"></a>聚合

```sql
aggregation:
    : GROUP BY expressions [WITH ROLLUP | WITH CUBE | GROUPING SETS (expressions)]
```

使用一个或多个聚合函数按一组表达式进行分组。 常见的内置聚合函数包括 count、avg、min、max 和 sum。

**`ROLLUP`**

在指定表达式的每个层次结构级别创建一个分组集。 例如，`GROUP BY a, b, c WITH ROLLUP` 等效于 `GROUP BY a, b, c GROUPING SETS ((a, b, c), (a, b), (a), ())`。 分组集的总数将是 `N + 1`，其中，`N` 是组表达式的数目。

**`CUBE`**

为指定表达式的集的每个可能组合创建一个分组集。 例如，`GROUP BY a, b, c WITH CUBE` 等效于 `GROUP BY a, b, c GROUPING SETS ((a, b, c), (a, b), (b, c), (a, c), (a), (b), (c), ())`。 分组集的总数将是 `2^N`，其中，`N` 是组表达式的数目。

**`GROUPING SETS`**

针对分组集中指定的组表达式的每个子集，执行分组依据操作。 例如，`GROUP BY x, y GROUPING SETS (x, y)` 等效于 `GROUP BY x` 的结果与 `GROUP BY y` 的结果的联合。

### <a name="examples"></a>示例

```sql
SELECT height, COUNT(*) AS num_rows FROM boxes GROUP BY height
SELECT width, AVG(length) AS average_length FROM boxes GROUP BY width
SELECT width, length, height FROM boxes GROUP BY width, length, height WITH ROLLUP
SELECT width, length, avg(height) FROM boxes GROUP BY width, length GROUPING SETS (width, length)
```

## <a name="window-functions"></a>开窗函数

```sql
window_expression:
  : expression OVER window_spec

named_window:
  : window_identifier AS window_spec

window_spec:
  | window_identifier
  : ( [PARTITION | DISTRIBUTE] BY expressions
        [[ORDER | SORT] BY sort_expressions] [window_frame])

window_frame:
  | [RANGE | ROWS] frame_bound
  : [RANGE | ROWS] BETWEEN frame_bound AND frame_bound

frame_bound:
  | CURRENT ROW
  | UNBOUNDED [PRECEDING | FOLLOWING]
  : expression [PRECEDING | FOLLOWING]
```

基于一系列输入行来计算结果。 窗口化表达式是使用 `OVER` 关键字指定的，后面跟有窗口的标识符（使用 `WINDOW` 关键字进行定义）或窗口的规范。

**`PARTITION BY`**

指定哪些行将位于同一分区中，其别名为 `DISTRIBUTE BY`。

**`ORDER BY`**

指定如何对窗口分区中的行排序，其别名为 `SORT BY`。

**`RANGE bound`**

将窗口的大小表示为表达式的值范围。

**`ROWS bound`**

将窗口的大小表示为当前行之前和/或之后的行数。

**`CURRENT ROW`**

使用当前行作为边界。

**`UNBOUNDED`**

使用负无穷作为下限，或使用无穷大作为上限。

**`PRECEDING`**

如果与 `RANGE` 边界一起使用，则这将定义值范围的下限。 如果与 `ROWS` 边界一起使用，则这将确定当前行之前要保留在窗口中的行数。

**`FOLLOWING`**

如果与 `RANGE` 边界一起使用，则这将定义值范围的上限。 如果与 `ROWS` 边界一起使用，则这将确定当前行之后要保留在窗口中的行数。

## <a name="hints"></a>提示

```sql
hints:
  : /*+ hint[, hint, ...] */
  hint:
    : hintName [(expression[, expression, ...])]
```

提示可用来帮助 Spark 更好地执行查询。 例如，你可以提示某个表足够小，可以[广播](https://spark.apache.org/docs/latest/sql-performance-tuning.html#broadcast-hint-for-sql-queries)，这将加快联接速度。
可以将一个或多个提示添加到 /*+ … */ 注释块内的 `SELECT` 语句。 可以在同一个注释块中指定多个提示，在这种情况下，提示以逗号分隔，并且可以有多个这样的注释块。 提示有一个名称（例如 `BROADCAST`），并接受 0 个或 0 个以上的参数。

### <a name="examples"></a>示例

```sql
SELECT /*+ BROADCAST(customers) */ * FROM customers, orders WHERE o_custId = c_custId
SELECT /*+ SKEW('orders') */ * FROM customers, orders WHERE o_custId = c_custId
SELECT /*+ SKEW('orders'), BROADCAST(demographic) */ * FROM orders, customers, demographic WHERE o_custId = c_custId AND c_demoId = d_demoId
```

（Azure Databricks 上的 Delta Lake）有关 `SKEW` 提示的详细信息，请参阅[倾斜联接优化](../../../../delta/join-performance/skew-join.md#skew-join)。