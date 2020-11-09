---
title: Kusto 对中间聚合结果进行分区和组合 - Azure 数据资源管理器
description: 本文介绍了如何在 Azure 数据资源管理器中对聚合的中间结果进行分区和组合。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 0ab588482a656e5d21d3eb59c9c253e1e0a45599
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106115"
---
# <a name="using-hll-and-tdigest"></a>使用 hll() 和 tdigest()

假设要计算过去 7 天内每日不同用户的计数。 你可以每天运行一次 `summarize dcount(user)`，并将时间跨度筛选为过去七天。 此方法效率较低，因为每次运行计算时与上一计算之间有六天的重叠。 你还可以计算每天的聚合，然后将这些聚合组合在一起。 此方法要求你“记住”最后六个结果，但效率要高得多。

如上所述的分区查询对于简单聚合（例如 `count()` 和 `sum()`）很容易。 它还可用于复杂聚合，例如 `dcount()` 和 `percentiles()`。 本主题说明了 Kusto 如何支持此类计算。

下面的示例展示了如何使用 `hll`/`tdigest`，表明在某些情况下使用这些命令是高效的：

> [!NOTE]
> 在某些情况下，`hll` 或 `tdigest` 聚合函数生成的动态对象可能会很大，会超出编码策略中的默认 MaxValueSize 属性值。 如果是这样，则该对象将作为 null 引入。
> 例如，在准确度为 4 的情况下保存 `hll` 函数的输出时，`hll` 对象的大小超出了默认的 MaxValueSize (1MB)。
> 若要避免此问题，请修改列的编码策略，如下例所示。

```kusto
range x from 1 to 1000000 step 1
| summarize hll(x,4)
| project sizeInMb = estimate_data_size(hll_x) / pow(1024,2)
```

|sizeInMb|
|---|
|1.0000524520874|

在应用此类策略之前将此对象引入到表中会引入 null：

```kusto
.set-or-append MyTable <| range x from 1 to 1000000 step 1
| summarize hll(x,4)
```

```kusto
MyTable
| project isempty(hll_x)
```

| Column1 |
|---------|
| 1       |

若要避免引入 null，请使用特殊编码策略类型 `bigobject`，它将 `MaxValueSize` 重写为 2 MB，如下所示：

```kusto
.alter column MyTable.hll_x policy encoding type='bigobject'
```

现在将值引入到上面的同一个表中：

```kusto
.set-or-append MyTable <| range x from 1 to 1000000 step 1
| summarize hll(x,4)
```

成功引入第二个值： 

```kusto
MyTable
| project isempty(hll_x)
```

|Column1|
|---|
|1|
|0|


## <a name="example-count-with-binned-timestamp"></a>示例：使用装箱时间戳计数

这里有一个表 `PageViewsHllTDigest`，其中包含每小时查看的页面的 `hll` 值。 你希望将这些值分段成 `12h`。 使用 `hll_merge()` 聚合函数合并 `hll` 值，将时间戳分段成 `12h`。 使用函数 `dcount_hll` 返回最终的 `dcount` 值：

```kusto
PageViewsHllTDigest
| summarize merged_hll = hll_merge(hllPage) by bin(Timestamp, 12h)
| project Timestamp , dcount_hll(merged_hll)
```

|Timestamp|`dcount_hll_merged_hll`|
|---|---|
|2016-05-01 12:00:00.0000000|20056275|
|2016-05-02 00:00:00.0000000|38797623|
|2016-05-02 12:00:00.0000000|39316056|
|2016-05-03 00:00:00.0000000|13685621|

将时间戳分段成 `1d`：

```kusto
PageViewsHllTDigest
| summarize merged_hll = hll_merge(hllPage) by bin(Timestamp, 1d)
| project Timestamp , dcount_hll(merged_hll)
```

|Timestamp|`dcount_hll_merged_hll`|
|---|---|
|2016-05-01 00:00:00.0000000|20056275|
|2016-05-02 00:00:00.0000000|64135183|
|2016-05-03 00:00:00.0000000|13685621|

可以对 `tdigest` 的值执行同一查询，这些值表示每小时的 `BytesDelivered`：

```kusto
PageViewsHllTDigest
| summarize merged_tdigests = merge_tdigests(tdigestBytesDel) by bin(Timestamp, 12h)
| project Timestamp , percentile_tdigest(merged_tdigests, 95, typeof(long))
```

|Timestamp|`percentile_tdigest_merged_tdigests`|
|---|---|
|2016-05-01 12:00:00.0000000|170200|
|2016-05-02 00:00:00.0000000|152975|
|2016-05-02 12:00:00.0000000|181315|
|2016-05-03 00:00:00.0000000|146817|
 
## <a name="example-temporary-table"></a>示例：临时表

如果数据集太大，则会达到 Kusto 限制。在这种情况下，你需要对数据集运行定期查询，但需要运行常规查询来针对大型数据集计算 [`percentile()`](percentiles-aggfunction.md) 或 [`dcount()`](dcount-aggfunction.md)。

::: zone pivot="azuredataexplorer"

若要解决此问题，可以将新增加的数据作为 `hll` 或 `tdigest` 值添加到一个临时表中，当所需的运算为 `dcount` 或 [`tdigest()`](tdigest-aggfunction.md) 时，请使用 [`hll()`](hll-aggfunction.md)；当所需的运算为 percentile 时，请使用 [`set/append`](../../ingest-data-overview.md) 或 [`update policy`](../management/updatepolicy.md)。 在这种情况下，`dcount` 或 `tdigest` 的中间结果会保存到另一个数据集中，该数据集应小于大的目标数据集。

::: zone-end

::: zone pivot="azuremonitor"

若要解决此问题，可以将新增加的数据作为 `hll` 或 `tdigest` 值添加到一个临时表中，当所需的运算为 `dcount` 时，请使用 [`hll()`](hll-aggfunction.md)。 在这种情况下，`dcount` 的中间结果保存到另一个数据集中，该数据集应小于大的目标数据集。

::: zone-end

当需要获取这些值的最终结果时，可以让查询使用 `hll`/`tdigest` 合并：[`hll-merge()`](hll-merge-aggfunction.md)/[`tdigest_merge()`](tdigest-merge-aggfunction.md)。 然后，在获取合并值之后，可以对这些合并值调用 [`percentile_tdigest()`](percentile-tdigestfunction.md) / [`dcount_hll()`](dcount-hllfunction.md)，以获取 `dcount` 或 percentile 运算的最终结果。

假设有一个表 PageViews，每天向其中引入数据。在 date = datetime(2016-05-01 18:00:00.0000000) 过后，你每天都要计算每分钟查看的页面的非重复计数。

运行以下查询：

```kusto
PageViews   
| where Timestamp > datetime(2016-05-01 18:00:00.0000000)
| summarize percentile(BytesDelivered, 90), dcount(Page,2) by bin(Timestamp, 1d)
```

|Timestamp|percentile_BytesDelivered_90|dcount_Page|
|---|---|---|
|2016-05-01 00:00:00.0000000|83634|20056275|
|2016-05-02 00:00:00.0000000|82770|64135183|
|2016-05-03 00:00:00.0000000|72920|13685621|

你每次运行此查询时（例如，你希望一天运行多次此查询），此查询都会聚合所有值。

如果你使用更新策略或 set/append 命令将 `hll` 和 `tdigest` 值（这些值是 `dcount` 和 percentile 运算的中间结果）保存到临时表 `PageViewsHllTDigest` 中，则可以仅合并这些值，然后通过以下查询使用 `dcount_hll`/`percentile_tdigest`：

```kusto
PageViewsHllTDigest
| summarize  percentile_tdigest(merge_tdigests(tdigestBytesDel), 90), dcount_hll(hll_merge(hllPage)) by bin(Timestamp, 1d)
```

|Timestamp|`percentile_tdigest_merge_tdigests_tdigestBytesDel`|`dcount_hll_hll_merge_hllPage`|
|---|---|---|
|2016-05-01 00:00:00.0000000|84224|20056275|
|2016-05-02 00:00:00.0000000|83486|64135183|
|2016-05-03 00:00:00.0000000|72247|13685621|

此查询的性能应当更高，因为它针对更小的表运行。 在此示例中，第一个查询针对大约 215M 记录运行，而第二个查询仅针对 32 条记录运行：

## <a name="example-intermediate-results"></a>示例：中间结果

保留查询。
假设你有一个表，其中汇总了用户查看每个维基百科页面的时间（样本大小为 10M），并且对于每个 date1 date2，你想要查明在 date1 和 date2 这两天查看的页面数相对于在 date1 (date1 < date2) 这天查看的页面数的比率。
  
琐细的方法是使用 join 和 summarize 运算符：

```kusto
// Get the total pages viewed each day
let totalPagesPerDay = PageViewsSample
| summarize by Page, Day = startofday(Timestamp)
| summarize count() by Day;
// Join the table to itself to get a grid where 
// each row shows foreach page1, in which two dates
// it was viewed.
// Then count the pages between each two dates to
// get how many pages were viewed between date1 and date2.
PageViewsSample
| summarize by Page, Day1 = startofday(Timestamp)
| join kind = inner
(
    PageViewsSample
    | summarize by Page, Day2 = startofday(Timestamp)
)
on Page
| where Day2 > Day1
| summarize count() by Day1, Day2
| join kind = inner
    totalPagesPerDay
on $left.Day1 == $right.Day
| project Day1, Day2, Percentage = count_*100.0/count_1
```

|第 1 天|第 2 天|百分比|
|---|---|---|
|2016-05-01 00:00:00.0000000|2016-05-02 00:00:00.0000000|34.0645725975255|
|2016-05-01 00:00:00.0000000|2016-05-03 00:00:00.0000000|16.618368960101|
|2016-05-02 00:00:00.0000000|2016-05-03 00:00:00.0000000|14.6291376489636|

 
以上查询用了大约 18 秒的时间。

使用 [`hll()`](hll-aggfunction.md)、[`hll_merge()`](hll-merge-aggfunction.md) 和 [`dcount_hll()`](dcount-hllfunction.md) 函数时，等效的查询将在大约 1.3 秒后结束，这表明 `hll` 函数将上述查询的速度提高了大约 13 倍：

```kusto
let Stats=PageViewsSample | summarize pagehll=hll(Page, 2) by day=startofday(Timestamp); // saving the hll values (intermediate results of the dcount values)
let day0=toscalar(Stats | summarize min(day)); // finding the min date over all dates.
let dayn=toscalar(Stats | summarize max(day)); // finding the max date over all dates.
let daycount=tolong((dayn-day0)/1d); // finding the range between max and min
Stats
| project idx=tolong((day-day0)/1d), day, pagehll
| mv-expand pidx=range(0, daycount) to typeof(long)
// Extend the column to get the dcount value from hll'ed values for each date (same as totalPagesPerDay from the above query)
| extend key1=iff(idx < pidx, idx, pidx), key2=iff(idx < pidx, pidx, idx), pages=dcount_hll(pagehll)
// For each two dates, merge the hll'ed values to get the total dcount over each two dates, 
// This helps to get the pages viewed in both date1 and date2 (see the description below about the intersection_size)
| summarize (day1, pages1)=arg_min(day, pages), (day2, pages2)=arg_max(day, pages), union_size=dcount_hll(hll_merge(pagehll)) by key1, key2
| where day2 > day1
// To get pages viewed in date1 and also date2, look at the merged dcount of date1 and date2, subtract it from pages of date1 + pages on date2.
| project pages1, day1,day2, intersection_size=(pages1 + pages2 - union_size)
| project day1, day2, Percentage = intersection_size*100.0 / pages1
```

|day1|day2|百分比|
|---|---|---|
|2016-05-01 00:00:00.0000000|2016-05-02 00:00:00.0000000|33.2298494510578|
|2016-05-01 00:00:00.0000000|2016-05-03 00:00:00.0000000|16.9773830213667|
|2016-05-02 00:00:00.0000000|2016-05-03 00:00:00.0000000|14.5160020350006|

> [!NOTE] 
> 由于 `hll` 函数的误差，查询结果不是 100% 准确。 有关误差的详细信息，请参阅 [`dcount()`](dcount-aggfunction.md)。
