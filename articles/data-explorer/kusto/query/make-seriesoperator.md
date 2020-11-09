---
title: make-series 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 make-series 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/16/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9fd3c35090ca2dd17c0fa0c4396897f62fdbc3cc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104585"
---
# <a name="make-series-operator"></a>make-series 运算符

沿指定的轴创建指定聚合值的序列。

```kusto
T | make-series sum(amount) default=0, avg(price) default=0 on timestamp from datetime(2016-01-01) to datetime(2016-01-10) step 1d by fruit, supplier
```

## <a name="syntax"></a>语法

T `| make-series` [MakeSeriesParamters] [Column `=`] Aggregation [`default` `=` DefaultValue] [`,` ...] `on` AxisColumn [`from` start] [`to` end] `step` step [`by` [Column `=`] GroupExpression [`,` ...]]          

## <a name="arguments"></a>参数

* *Column* ：结果列的可选名称。 默认为派生自表达式的名称。
* DefaultValue：将使用默认值，而不是不存在的值。 如果没有任何行包含特定的 AxisColumn 和 GroupExpression 值，则在结果中，将为数组的相应元素分配 DefaultValue  。 如果省略 DefaultValue，则假定为 0。 
* *聚合：* 对 [聚合函数](make-seriesoperator.md#list-of-aggregation-functions)（如 `count()` 或 `avg()`）的调用，使用列名作为参数。 请参阅[聚合函数列表](make-seriesoperator.md#list-of-aggregation-functions)。 只能将返回数值结果的聚合函数与 `make-series` 运算符配合使用。
* AxisColumn：将用作序列排序依据的列。 可将其视为时间线，但接受除 `datetime` 之外的任何数字类型。
* start：（可选）要生成的每个序列的 AxisColumn 下限值 。 start、end 以及 step 用于生成由给定范围内使用指定 step 的 AxisColumn 值组成的数组    。 所有 Aggregation 值分别按顺序排列到此数组。 此 AxisColumn 数组也是输出中与 AxisColumn 同名的最后一个输出列 。 如果未指定 start 值，则其是每个序列中包含数据的第一个 bin (step)。
* 结束：（可选）AxisColumn 的上限（不含）值 。 时序的最后一个索引小于此值（并且将是小于 end 的以下值：start 加上 step 的整数倍）  。 如果未提供 end 值，则其将是每个序列包含数据的最后一个 bin (step) 的上限。
* step：AxisColumn 数组中两个连续元素之间的差异（即 bin 大小）。
* GroupExpression：各列的表达式，提供一组非重复值。 通常，它是已提供一组受限值的列名。 
* MakeSeriesParameters：零个或更多（以空格分隔）Name `=` Value 形式的参数，用于控制行为 。 支持以下参数： 
  
  |名称           |值                                        |说明                                                                                        |
  |---------------|-------------------------------------|------------------------------------------------------------------------------|
  |`kind`          |`nonempty`                               |当 make-series 运算符的输入为空时，生成默认结果|                                

## <a name="returns"></a>返回

输入行将排列成与 `by` 表达式以及 `bin_at(`AxisColumn`, `step`, `start`)` 表达式具有相同值的组  。 然后，对每个组计算指定的聚合函数，从而为每组生成行。 结果包含 `by` 列和 AxisColumn 列，还至少包含用于每个计算聚合的一列。 （不支持聚合多个列或非数值结果。）

此中间结果包含的行数与 `by` 和 `bin_at(`AxisColumn`, `step`, `start`)` 值的不同组合数相同  。

最后，中间结果中的行被排列为具有相同 `by` 表达式值的组，且所有聚合值都排列为数组（`dynamic` 类型的值）。 对于每个聚合，都有一列包含其名称相同的数组。 范围函数的输出中包含所有 AxisColumn 值的最后一列。 对于所有行，其值都重复。 

由于使用默认值时将填充缺失的 bin，因此生成的数据透视表中所有序列具有相同的 bin 数（即聚合值）  

**注意**

尽管可为聚合和分组表达式提供任意表达式，但使用简单的列名更加高效。

**替代语法**

T `| make-series` [Column `=`] Aggregation [`default` `=` DefaultValue] [`,` ...] `on` AxisColumn `in` `range(`start`,` stop`,` step`)` [`by` [Column `=`] GroupExpression [`,` ...]]         

使用替代语法生成的序列与使用主语法生成的序列有两个方面的差异：
* stop 值包含在内。
* 使用 bin() 而不是 bin_at() 生成索引轴分箱，这意味着生成的序列中可能不包括 start。

建议使用 make-series 的主要语法，而不是替代语法。

**分布和随机选择**

`make-series` 支持使用语法 hint.shufflekey `summarize` [shufflekey hints](shufflequery.md)。

## <a name="list-of-aggregation-functions"></a>聚合函数列表

|函数|描述|
|--------|-----------|
|[any()](any-aggfunction.md)|返回组的随机非空值|
|[avg()](avg-aggfunction.md)|返回整个组的平均值|
|[avgif()](avgif-aggfunction.md)|返回具有组谓词的平均值|
|[count()](count-aggfunction.md)|返回组的计数|
|[countif()](countif-aggfunction.md)|返回具有组谓词的计数|
|[dcount()](dcount-aggfunction.md)|返回组元素的近似非重复计数|
|[dcountif()](dcountif-aggfunction.md)|返回具有组谓词的近似非重复计数|
|[max()](max-aggfunction.md)|返回组内的最大值|
|[maxif()](maxif-aggfunction.md)|返回具有组谓词的最大值|
|[min()](min-aggfunction.md)|返回组内的最小值|
|[minif()](minif-aggfunction.md)|返回具有组谓词的最小值|
|[stdev()](stdev-aggfunction.md)|返回整个组的标准偏差|
|[sum()](sum-aggfunction.md)|返回组中元素的总和|
|[sumif()](sumif-aggfunction.md)|返回具有组谓词的元素的总和|
|[variance()](variance-aggfunction.md)|返回整个组的方差|

## <a name="list-of-series-analysis-functions"></a>序列分析函数列表

|函数|描述|
|--------|-----------|
|[series_fir()](series-firfunction.md)|应用有限脉冲响应滤波器|
|[series_iir()](series-iirfunction.md)|应用无限脉冲响应滤波器|
|[series_fit_line()](series-fit-linefunction.md)|查找与输入最近似的直线|
|[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)|查找与输入最近似的线，返回动态对象|
|[series_fit_2lines()](series-fit-2linesfunction.md)|查找与输入最近似的两条线|
|[series_fit_2lines_dynamic()](series-fit-2lines-dynamicfunction.md)|查找与输入最近似的两条线，返回动态对象|
|[series_outliers()](series-outliersfunction.md)|对序列中的异常点进行评分|
|[series_periods_detect()](series-periods-detectfunction.md)|找出一个时序中最重要的周期|
|[series_periods_validate()](series-periods-validatefunction.md)|检查时序是否包含给定长度的定期模式|
|[series_stats_dynamic()](series-stats-dynamicfunction.md)|返回包含常用统计信息（最小值/最大值/方差/标准偏差/平均值）的多个列|
|[series_stats()](series-statsfunction.md)|生成包含常用统计信息（最小值/最大值/方差/标准偏差/平均值）的动态值|
  
## <a name="list-of-series-interpolation-functions"></a>序列内插函数列表

|函数|描述|
|--------|-----------|
|[series_fill_backward()](series-fill-backwardfunction.md)|在序列中对缺失值执行后向填充内插|
|[series_fill_const()](series-fill-constfunction.md)|用指定的常数值替换序列中缺失的值|
|[series_fill_forward()](series-fill-forwardfunction.md)|在序列中对缺失值执行前向填充内插|
|[series_fill_linear()](series-fill-linearfunction.md)|在序列中对缺失值执行线性内插|

* 注意：默认情况下，内插函数假定 `null` 为缺失值。 因此，如果要对序列使用内插函数，请在 `make-series` 中指定 `default=`double(`null`)。 

## <a name="example"></a>示例
  
 一张表，其中显示了从每家供应商订购的每种水果的数量和平均价格组成的数组，该表的排序依据是指定范围的时间戳。 水果与供应商的每个不同组合在输出中都占一行。 输出列显示水果、供应商，以及由以下元素组成的数组：计数、平均值和整个时间线 (2016-01-01 - 2016-01-10)。 所有数组都按各自的时间戳排序，并且所有间隙均用默认值（在本示例中为 0）填充。 忽略所有其他输入列。
  
```kusto
T | make-series PriceAvg=avg(Price) default=0
on Purchase from datetime(2016-09-10) to datetime(2016-09-13) step 1d by Supplier, Fruit
```

:::image type="content" source="images/make-seriesoperator/makeseries.png" alt-text="三个表。第一个表列出了原始数据，第二个表只有不同的供应商-水果-日期组合，第三个表包含 make-series 结果。":::  

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples --> 
```kusto
let data=datatable(timestamp:datetime, metric: real)
[
  datetime(2016-12-31T06:00), 50,
  datetime(2017-01-01), 4,
  datetime(2017-01-02), 3,
  datetime(2017-01-03), 4,
  datetime(2017-01-03T03:00), 6,
  datetime(2017-01-05), 8,
  datetime(2017-01-05T13:40), 13,
  datetime(2017-01-06), 4,
  datetime(2017-01-07), 3,
  datetime(2017-01-08), 8,
  datetime(2017-01-08T21:00), 8,
  datetime(2017-01-09), 2,
  datetime(2017-01-09T12:00), 11,
  datetime(2017-01-10T05:00), 5,
];
let interval = 1d;
let stime = datetime(2017-01-01);
let etime = datetime(2017-01-10);
data
| make-series avg(metric) on timestamp from stime to etime step interval 
```
  
|avg_metric|timestamp|
|---|---|
|[ 4.0, 3.0, 5.0, 0.0, 10.5, 4.0, 3.0, 8.0, 6.5 ]|[ "2017-01-01T00:00:00.0000000Z", "2017-01-02T00:00:00.0000000Z", "2017-01-03T00:00:00.0000000Z", "2017-01-04T00:00:00.0000000Z", "2017-01-05T00:00:00.0000000Z", "2017-01-06T00:00:00.0000000Z", "2017-01-07T00:00:00.0000000Z", "2017-01-08T00:00:00.0000000Z", "2017-01-09T00:00:00.0000000Z" ]|  


当 `make-series` 的输入为空时，`make-series` 的默认行为也会生成一个空结果。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let data=datatable(timestamp:datetime, metric: real)
[
  datetime(2016-12-31T06:00), 50,
  datetime(2017-01-01), 4,
  datetime(2017-01-02), 3,
  datetime(2017-01-03), 4,
  datetime(2017-01-03T03:00), 6,
  datetime(2017-01-05), 8,
  datetime(2017-01-05T13:40), 13,
  datetime(2017-01-06), 4,
  datetime(2017-01-07), 3,
  datetime(2017-01-08), 8,
  datetime(2017-01-08T21:00), 8,
  datetime(2017-01-09), 2,
  datetime(2017-01-09T12:00), 11,
  datetime(2017-01-10T05:00), 5,
];
let interval = 1d;
let stime = datetime(2017-01-01);
let etime = datetime(2017-01-10);
data
| limit 0
| make-series avg(metric) default=1.0 on timestamp from stime to etime step interval 
| count 
```

|计数|
|---|
|0|


在 `make-series` 中使用 `kind=nonempty` 将生成默认值的非空结果：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let data=datatable(timestamp:datetime, metric: real)
[
  datetime(2016-12-31T06:00), 50,
  datetime(2017-01-01), 4,
  datetime(2017-01-02), 3,
  datetime(2017-01-03), 4,
  datetime(2017-01-03T03:00), 6,
  datetime(2017-01-05), 8,
  datetime(2017-01-05T13:40), 13,
  datetime(2017-01-06), 4,
  datetime(2017-01-07), 3,
  datetime(2017-01-08), 8,
  datetime(2017-01-08T21:00), 8,
  datetime(2017-01-09), 2,
  datetime(2017-01-09T12:00), 11,
  datetime(2017-01-10T05:00), 5,
];
let interval = 1d;
let stime = datetime(2017-01-01);
let etime = datetime(2017-01-10);
data
| limit 0
| make-series kind=nonempty avg(metric) default=1.0 on timestamp from stime to etime step interval 
```

|avg_metric|timestamp|
|---|---|
|[<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0<br>]|[<br>  "2017-01-01T00:00:00.0000000Z",<br>  "2017-01-02T00:00:00.0000000Z",<br>  "2017-01-03T00:00:00.0000000Z",<br>  "2017-01-04T00:00:00.0000000Z",<br>  "2017-01-05T00:00:00.0000000Z",<br>  "2017-01-06T00:00:00.0000000Z",<br>  "2017-01-07T00:00:00.0000000Z",<br>  "2017-01-08T00:00:00.0000000Z",<br>  "2017-01-09T00:00:00.0000000Z"<br>]|
