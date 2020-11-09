---
title: percentile()、percentiles() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 percentile()、percentiles()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/30/2020
ms.date: 10/29/2020
ms.openlocfilehash: eb93b639aa938604fd4d5dabc064d370227aa51d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105042"
---
# <a name="percentile-percentiles-aggregation-function"></a>percentile()、percentiles()（聚合函数）

返回由 `*Expr*` 定义的群体的指定[最接近排名百分位](#nearest-rank-percentile)的估计值。
准确性取决于百分位区域中的填充密度。 此函数只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用。

* `percentiles()` 与 `percentile()` 类似，但可以计算大量百分位值，这比分别计算每个百分位更快。
* `percentilesw()` 与 `percentilew()` 类似，但可以计算大量加权百分位值，这比分别计算每个百分位更快。
* `percentilew()` 和 `percentilesw()` 允许你计算加权百分位。 加权百分位按“加权”方式计算给定的百分位，方法是将每个值视为在输入中重复 `weight` 次。

## <a name="syntax"></a>语法

summarize `percentile(`*Expr*`,` *Percentile*`)`

summarize `percentiles(`*Expr*`,` *Percentile1* [`,` *Percentile2* ]`)`

summarize `percentiles_array(`*Expr*`,` *Percentile1* [`,` *Percentile2* ]`)`

summarize `percentiles_array(`*Expr*`,` *Dynamic array*`)`

summarize `percentilew(`*Expr*`,` *WeightExpr*`,` *Percentile*`)`

summarize `percentilesw(`*Expr*`,` *WeightExpr*`,` *Percentile1* [`,` *Percentile2* ]`)`

summarize `percentilesw_array(`*Expr*`,` *WeightExpr*`,` *Percentile1* [`,` *Percentile2* ]`)`

summarize `percentilesw_array(`*Expr*`,` *WeightExpr*`,` *Dynamic array*`)`

## <a name="arguments"></a>参数

* `*Expr*`：用于聚合计算的表达式。
* `*WeightExpr*`：表达式，将用作进行聚合计算的值的权重。
* `*Percentile*`：一个双精度常数，用于指定百分位。
* `*Dynamic array*`：整数或浮点数动态数组中的百分位列表。

## <a name="returns"></a>返回

返回组中指定百分位的 `*Expr*` 的估计值。 

## <a name="examples"></a>示例

大于示例集 95% 和小于示例集 5% 的 `Duration` 值。

```kusto
CallDetailRecords | summarize percentile(Duration, 95) by continent
```

同时计算 5、50（中值）和 95。

```kusto
CallDetailRecords 
| summarize percentiles(Duration, 5, 50, 95) by continent
```

:::image type="content" source="images/percentiles-aggfunction/percentiles.png" alt-text="表格中列出了结果，其中包含洲的列以及第五、第五十和第九十五百分位数的持续时间值的列。":::

结果表明，在欧洲，5% 的调用短于 11.55 秒，50% 的调用短于 3 分钟 18.46 秒，95% 的调用短于 40 分钟 48 秒。

```kusto
CallDetailRecords 
| summarize percentiles(Duration, 5, 50, 95), avg(Duration)
```

## <a name="weighted-percentiles"></a>加权百分位数

假设你重复测量完成某个操作所需的时间（持续时间）。 你将记录每个持续时间值（舍入到 100 毫秒）和舍入值的出现次数 (BucketSize)，而非记录每个测量值。

使用 `summarize percentilesw(Duration, BucketSize, ...)` 按“加权”方式计算给定的百分位。 将持续时间的每个值视为在输入中重复 BucketSize 次，实际上不需要实现这些记录。

## <a name="example"></a>示例

客户有一组延迟值（以毫秒为单位）：`{ 1, 1, 2, 2, 2, 5, 7, 7, 12, 12, 15, 15, 15, 18, 21, 22, 26, 35 }`。

若要减少带宽和存储，请对以下 Bucket 进行预聚合：`{ 10, 20, 30, 40, 50, 100 }`。 计算每个 Bucket 中的事件数以生成下表：

:::image type="content" source="images/percentiles-aggfunction/percentilesw-table.png" alt-text="Percentilesw 表":::

该表：
 * 在 10 毫秒 Bucket 内显示了 8 个事件（对应于子集 `{ 1, 1, 2, 2, 2, 5, 7, 7 }`）
 * 在 20 毫秒 Bucket 内显示了 6 个事件（对应于子集 `{ 12, 12, 15, 15, 15, 18 }`）
 * 在 30 毫秒 Bucket 内显示了 3 个事件（对应于子集 `{ 21, 22, 26 }`）
 * 在 40 毫秒 Bucket 内显示了 1 个事件（对应于子集 `{ 35 }`）

此时，原始数据不再可用。 仅提供每个 Bucket 中的事件数。 若要根据此数据计算百分位，请使用 `percentilesw()` 函数。
例如，对于 50、75 和 99.9 百分位，请使用以下查询。

```kusto
datatable (ReqCount:long, LatencyBucket:long) 
[ 
    8, 10, 
    6, 20, 
    3, 30, 
    1, 40 
]
| summarize percentilesw(LatencyBucket, ReqCount, 50, 75, 99.9) 
```

上述查询的结果是：

:::image type="content" source="images/percentiles-aggfunction/percentilesw-result.png" alt-text="Percentilesw 结果" border="false":::


如果将数据扩展到以下形式，则上述查询对应于函数 `percentiles(LatencyBucket, 50, 75, 99.9)`：

:::image type="content" source="images/percentiles-aggfunction/percentilesw-rawtable.png" alt-text="Percentilesw 原始表":::

## <a name="getting-multiple-percentiles-in-an-array"></a>在数组中获取多个百分位

可以在单个动态列中以数组形式获取多个百分位，而不是在多个列中获取。

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, 5, 25, 50, 75, 95), avg(Duration)
```

:::image type="content" source="images/percentiles-aggfunction/percentiles-array-result.png" alt-text="百分位数组结果":::

同样，可以使用 `percentilesw_array` 将加权百分位作为动态数组返回。

可以在整数或浮点数动态数组中指定 `percentiles_array` 和 `percentilesw_array` 的百分位。 数组必须是常数，但不一定是文本。

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, dynamic([5, 25, 50, 75, 95])), avg(Duration)
```

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, range(0, 100, 5)), avg(Duration)
```

## <a name="nearest-rank-percentile"></a>最接近排名百分位

排序值（按从小到大排序）的列表中第 *P* 个百分位 (0 < *P* <= 100) 是列表中的最小值。 数据的 P% 小于或等于第 P 个百分位值（请参考维基百科上关于百分位的文章）。

将第 0 个百分位定义为群体的最小成员。

>[!NOTE]
> 由于计算的近似性质，实际返回的值可能不是群体的成员。
> 最接近排名定义意味着  P= 50 不符合中间值的插值定义。 针对特定应用程序计算此差异的重要性时，应考虑群体大小和[估算误差](#estimation-error-in-percentiles)。

## <a name="estimation-error-in-percentiles"></a>百分位中的估算错误

聚合百分位使用 [T-Digest](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf) 提供近似值。

>[!NOTE]
> * 估算错误的界限因请求百分位的值而异。 在 [0..100] 范围的两端，准确性最高。 0 和 100 百分位是分布的确切最小值和最大值。 越接近此范围的中间值，准确性越低。 中间值准确性最差，不超过 1%。
> * 错误边界通过级别观察而不是通过值。 假设 percentile(X, 50) 返回了值 Xm。 此估算可保证至少 49%，至多 51% 的 X 值小于或等于 Xm。 对 Xm 与 X 的实际中间值之间的差异没有理论上的限制。
> * 估算有时可能会得出精确的值，但没有可靠的条件来定义何时会出现这种情况。
