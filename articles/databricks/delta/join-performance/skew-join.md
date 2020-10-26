---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 倾斜联接优化 - Azure Databricks
description: 了解 Azure Databricks 上的 Delta Lake 如何优化带倾斜的表的联接性能。
ms.openlocfilehash: 8f9a6fc3a42f2e92520f8c0b4212fbfef6ca563a
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121844"
---
# <a name="skew-join-optimization"></a><a id="skew-join"> </a><a id="skew-join-optimization"> </a>倾斜联接优化

数据倾斜是指表的数据不均匀地分布在群集的多个分区中的一种情况。 数据偏斜可能会严重降低查询（尤其是那些使用联接的查询）的性能。 大表之间的联接需要混排数据，而倾斜可能导致群集中的工作极其不平衡。 如果一个查询在完成很少的任务（例如，200 个任务中的最后 3 个任务）时停滞不前，那么很可能是数据倾斜在影响该查询。 要验证：

1. 单击停滞的阶段，验证它是否正在进行联接。
2. 查询完成后，请找到执行联接的阶段并检查任务持续时间分布。
3. 通过减少持续时间对任务进行排序，并检查前几个任务。 如果一个任务完成的时间比其他任务长得多，则存在倾斜。

为了减轻倾斜，Azure Databricks 上的 Delta Lake SQL 会接受查询中的倾斜提示。 使用这些提示中的信息，Spark 可以构造更好的查询计划，该计划不会受到数据倾斜的影响。

## <a name="only-relation-name"></a>仅关系名称

倾斜提示必须至少包含具有倾斜的关系的名称。 关系是表、视图或子查询。
然后，与此关系进行的所有联接都会使用倾斜联接优化。

```sql
-- table with skew
SELECT /*+ SKEW('orders') */ * FROM orders, customers WHERE c_custId = o_custId

-- subquery with skew
SELECT /*+ SKEW('C1') */ *
  FROM (SELECT * FROM customers WHERE c_custId < 100) C1, orders
  WHERE C1.c_custId = o_custId
```

## <a name="relation-and-columns"></a>关系和列

一个关系可能有多个联接，只有其中一部分会受到倾斜的影响。
倾斜联接优化有一些开销，因此最好是只在需要时使用它。
倾斜提示会为此接受列名。 只有使用这些列的联接才使用倾斜联接优化。

```sql
-- single column
SELECT /*+ SKEW('orders', 'o_custId') */ *
  FROM orders, customers
  WHERE o_custId = c_custId

-- multiple columns
SELECT /*+ SKEW('orders', ('o_custId', 'o_storeRegionId')) */ *
  FROM orders, customers
  WHERE o_custId = c_custId AND o_storeRegionId = c_regionId
```

## <a name="relation-columns-and-skew-values"></a>关系、列和倾斜值

你还可以在提示中指定倾斜值。 倾斜值可能是已知的值（例如，是从不改变的值），也可能是很容易找到的值，具体取决于查询和数据。这样做可以减少倾斜联接优化的开销。 如果不这样做，则 Delta Lake 会自动检测这些值。

```sql
-- single column, single skew value
SELECT /*+ SKEW('orders', 'o_custId', 0) */ *
  FROM orders, customers
  WHERE o_custId = c_custId

-- single column, multiple skew values
SELECT /*+ SKEW('orders', 'o_custId', (0, 1, 2)) */ *
  FROM orders, customers
  WHERE o_custId = c_custId

-- multiple columns, multiple skew values
SELECT /*+ SKEW('orders', ('o_custId', 'o_storeRegionId'), ((0, 1001), (1, 1002))) */ *
  FROM orders, customers
  WHERE o_custId = c_custId AND o_storeRegionId = c_regionId
```