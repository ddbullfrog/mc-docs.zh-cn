---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 范围联接优化 - Azure Databricks
description: 了解使用间隔中的点或间隔重叠条件联接两个关系时，Azure Databricks 上的 Delta Lake 如何优化联接性能。
ms.openlocfilehash: d556e2e4232c3fadf4ec98cbc1d665ab343f4a36
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121843"
---
# <a name="range-join-optimization"></a>范围联接优化

当使用间隔中的点或间隔重叠条件联接两个关系时，将发生“范围联接”。
Databricks Runtime 中的范围联接优化支持可以在查询性能方面带来数量级的改进，但需要仔细地进行手动优化。

## <a name="point-in-interval-range-join"></a>间隔中的点范围联接

“间隔中的点范围联接”是一种联接，其中的条件包含的谓词指定一个关系中的值介于另一个关系中的两个值之间。 例如： 。

```sql
-- using BETWEEN expressions
SELECT *
FROM points JOIN ranges ON points.p BETWEEN ranges.start and ranges.end;

-- using inequality expressions
SELECT *
FROM points JOIN ranges ON points.p >= ranges.start AND points.p < ranges.end;

-- with fixed length interval
SELECT *
FROM points JOIN ranges ON points.p >= ranges.start AND points.p < ranges.start + 100;

-- join two sets of point values within a fixed distance from each other
SELECT *
FROM points1 p1 JOIN points2 p2 ON p1.p >= p2.p - 10 AND p1.p <= p2.p + 10;

-- a range condition together with other join conditions
SELECT *
FROM points, ranges
WHERE points.symbol = ranges.symbol
  AND points.p >= ranges.start
  AND points.p < ranges.end;
```

## <a name="interval-overlap-range-join"></a>间隔重叠范围联接

“间隔重叠范围联接”是一种联接，其中的条件包含的谓词指定每个关系中两个值之间的间隔的重叠。 例如： 。

```sql
-- overlap of [r1.start, r1.end] with [r2.start, r2.end]
SELECT *
FROM r1 JOIN r2 ON r1.start < r2.end AND r2.start < r1.end;

-- overlap of fixed length intervals
SELECT *
FROM r1 JOIN r2 ON r1.start < r2.start + 100 AND r2.start < r1.start + 100;

-- a range condition together with other join conditions
SELECT *
FROM r1 JOIN r2 ON r1.symbol = r2.symbol
  AND r1.start <= r2.end
  AND r1.end >= r2.start;
```

## <a name="range-join-optimization"></a>范围联接优化

范围联接优化针对满足以下条件的联接执行：

* 有一个条件可解释为间隔中的点范围联接或间隔重叠范围联接。
* 范围联接条件中涉及的所有值均为数值类型（整型、浮点、小数）、`DATE` 或 `TIMESTAMP`。
* 范围联接条件中涉及的所有值都属于同一类型。 对于十进制类型，这些值还需要具有相同的刻度和精度。
* 这是一个 `INNER JOIN`，或者，在间隔中的点范围联接中，这是在左侧具有点值的 `LEFT OUTER JOIN`，或者是在右侧具有点值的 `RIGHT OUTER JOIN`。
* 具有箱大小优化参数。

### <a name="bin-size"></a>箱大小

“箱大小”是一个数值优化参数，它将范围条件的值域拆分为多个大小相等的“箱”。 例如，如果箱大小为 10，则优化会将域拆分为长度为 10 的间隔的箱。
如果你在范围条件 `p BETWEEN start AND end` 中有一个点，并且 `start` 为 8，`end` 为 22，则此值间隔与长度为 10 的三个箱重叠 – 第一个箱从 0 到 10，第二个箱从 10 到 20，第三个箱从 20 到 30。 只有位于相同的三个箱中的点需要被视为该间隔的可能的联接匹配项。 例如，如果 `p` 为 32，则可以排除它位于 `start` 为 8 且 `end` 为 22 的范围中，因为它位于从 30 到 40 的箱中。

> [!NOTE]
>
> * 对于 `DATE` 值，箱大小的值被解释为天数。 例如，箱大小值为 7 表示一周。
> * 对于 `TIMESTAMP` 值，箱大小的值被解释为秒数。 如果需要亚秒值，则可以使用小数值。 例如，箱大小值为 60 表示一分钟，而箱大小值为 0.1 表示 100 毫秒。

你可以通过在查询中使用范围联接提示或通过设置会话配置参数来指定箱大小。
仅当手动指定箱大小时，才会应用范围联接优化。 [选择箱大小](#id2)部分介绍了如何选择最佳箱大小。

## <a name="enable-range-join-using-a-range-join-hint"></a>使用范围联接提示启用范围联接

若要在 SQL 查询中启用范围联接优化，可以使用范围联接提示来指定箱大小。
提示必须包含所联接的关系之一的关系名称和箱大小数值参数。
关系名称可以是表、视图或子查询。

```sql
SELECT /*+ RANGE_JOIN(points, 10) */ *
FROM points JOIN ranges ON points.p >= ranges.start AND points.p < ranges.end;

SELECT /*+ RANGE_JOIN(r1, 0.1) */ *
FROM (SELECT * FROM ranges WHERE ranges.amount < 100) r1, ranges r2
WHERE r1.start < r2.start + 100 AND r2.start < r1.start + 100;

SELECT /*+ RANGE_JOIN(c, 500) */ *
FROM a
  JOIN b ON (a.b_key = b.id)
  JOIN c ON (a.ts BETWEEN c.start_time AND c.end_time)
```

> [!NOTE]
>
> 在第三个示例中，你必须在 `c` 上放置提示。
> 这是因为联接是左结合，因此，该查询将被解释为 `(a JOIN b) JOIN c`，`a` 上的提示应用于 `a` 与 `b` 的联接，而不是与 `c` 的联接。

你还可以在某个已联接的数据帧上放置范围联接提示。 在这种情况下，提示仅包含箱大小数值参数。

```scala
val df1 = spark.table("ranges").as("left")
val df2 = spark.table("ranges").as("right")

val joined = df1.hint("range_join", 10)
  .join(df2, $"left.type" === $"right.type" &&
     $"left.end" > $"right.start" &&
     $"left.start" < $"right.end")

val joined2 = df1
  .join(df2.hint("range_join", 0.5), $"left.type" === $"right.type" &&
     $"left.end" > $"right.start" &&
     $"left.start" < $"right.end")
```

## <a name="enable-range-join-using-session-configuration"></a>使用会话配置启用范围联接

如果你不想修改查询，则可以将箱大小指定为配置参数。

```sql
SET spark.databricks.optimizer.rangeJoin.binSize=5
```

此配置适用于具有范围条件的任何联接。 但是，通过范围联接提示设置的不同箱大小始终会覆盖通过配置设置的大小。

## <a name="choose-the-bin-size"></a><a id="choose-the-bin-size"> </a><a id="id2"> </a>选择箱大小

范围联接优化的有效性取决于选择的箱大小是否合适。

箱大小越小，箱数量越多，这有助于筛选潜在的匹配项。
但是，如果箱大小显著小于所遇的值间隔，并且值间隔与多个箱间隔重叠，则会降低效率。 例如，如果条件为 `p BETWEEN start AND end`，其中 `start` 为 1,000,000 且 `end` 为 1,999,999，箱大小为 10，则值间隔与 100,000 个箱重叠。

如果间隔的长度比较统一并且是已知的，则建议你将箱大小设置为值间隔的典型预期长度。 但是，如果间隔的长度是变化且扭曲的，则必须设置一个能有效过滤短间隔的箱大小，同时防止长间隔重叠过多的箱，这就需要进行平衡。假设有一个 `ranges` 表，且 `start` 和 `end` 列之间存在间隔，则可使用以下查询确定扭曲间隔长度值的不同百分位数：

```sql
SELECT APPROX_PERCENTILE(CAST(end - start AS DOUBLE), ARRAY(0.5, 0.9, 0.99, 0.999, 0.9999)) FROM ranges
```

建议设置的箱大小为第 90 个百分位的值的最大值，或第 99 个百分位的值除以 10，或第 99.9 个百分位的值除以 100，依此类推。 基本原理是：

* 如果第 90 个百分位的值是箱大小，则只有 10% 的值间隔长度大于箱间隔，因此将跨 2 个以上的相邻箱间隔。
* 如果第 99 个百分位的值是箱大小，则只有 1% 的值间隔长度跨 11 个以上的相邻箱间隔。
* 如果第 99.9 个百分位的值是箱大小，则只有 0.1% 的值间隔长度跨 101 个以上的相邻箱间隔。
* 如果需要，可以针对第 99.99 个百分位、第 99.999 个百分位等百分位的值重复相同的步骤。

上述方法限制了与多个箱间隔重叠的扭曲长值间隔的数量。
这样获得的箱大小值只是进行优调的起点，实际结果可能取决于具体的工作负荷。