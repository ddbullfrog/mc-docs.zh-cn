---
title: summarize 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 summarize 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/20/2020
ms.date: 09/24/2020
ms.openlocfilehash: 4f4167b2433c14c6e52f57ed0c04219f1c448fa7
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146779"
---
# <a name="summarize-operator"></a>summarize 运算符

生成可聚合输入表内容的表。

```kusto
Sales | summarize NumTransactions=count(), Total=sum(UnitPrice * NumUnits) by Fruit, StartOfMonth=startofmonth(SellDateTime)
```

返回一个表，其中包含销售交易数目和每种水果每个销售月份的总金额。
输出列显示交易数目、交易价值、水果、以及记录了交易的月份开始时的日期/时间。

```kusto
T | summarize count() by price_range=bin(price, 10.0)
```

此表显示每个价格区间（[0,10.0]、[10.0,20.0] 等）内的项数。 此示例中有一列表示计数，还有一列表示价格范围。 忽略所有其他输入列。

## <a name="syntax"></a>语法

*T* `| summarize` [[*Column* `=`] *Aggregation* [`,` ...]] [`by` [*Column* `=`] *GroupExpression* [`,` ...]]

## <a name="arguments"></a>参数

* *Column*：结果列的可选名称。 默认为派生自表达式的名称。
* *聚合：* 对[聚合函数](summarizeoperator.md#list-of-aggregation-functions)（例如 `count()` 或 `avg()`）的调用，以列名作为参数。 请参阅[聚合函数的列表](summarizeoperator.md#list-of-aggregation-functions)。
* GroupExpression：一个可以引用输入数据的标量表达式。
  所有组表达式有多少个不同的值，输出就会包含多少个记录。

> [!NOTE]
> 当输入表为空时，输出取决于是否使用了 GroupExpression：
>
> * 如果未提供 GroupExpression，则输出将为单个（空）行。
> * 如果提供了 GroupExpression，则输出将不包含任何行。

## <a name="returns"></a>返回

输入行将排列成与 `by` 表达式具有相同值的组。 然后，对每个组计算指定的聚合函数，从而为每组生成行。 结果包含 `by` 列，还至少包含用于每个计算聚合的一列。 （某些聚合函数返回多个列。）

结果有许多行，因为 `by` 值（可能为零）存在不同的组合。 如果未提供任何组键，则结果将包含单个记录。

若要基于数值范围进行汇总，请使用 `bin()` 将范围减小为离散值。

> [!NOTE]
> * 尽管可为聚合和分组表达式提供任意表达式，但使用简单列名称或将 `bin()` 应用于数值列会更加高效。
> * 不再支持自动地每小时对日期/时间列进行分箱。 请改用显式分箱。 例如 `summarize by bin(timestamp, 1h)`。

## <a name="list-of-aggregation-functions"></a>聚合函数的列表

|函数|描述|
|--------|-----------|
|[any()](any-aggfunction.md)|返回组的随机非空值|
|[anyif()](anyif-aggfunction.md)|返回组的随机非空值（带谓词）|
|[arg_max()](arg-max-aggfunction.md)|当参数最大化时返回一个或多个表达式|
|[arg_min()](arg-min-aggfunction.md)|当参数最小化时返回一个或多个表达式|
|[avg()](avg-aggfunction.md)|返回整个组的平均值|
|[avgif()](avgif-aggfunction.md)|返回整个组的平均值（带谓词）|
|[binary_all_and](binary-all-and-aggfunction.md)|返回使用组的二元 `AND` 进行聚合的值|
|[binary_all_or](binary-all-or-aggfunction.md)|返回使用组的二元 `OR` 进行聚合的值|
|[binary_all_xor](binary-all-xor-aggfunction.md)|返回使用组的二元 `XOR` 进行聚合的值|
|[buildschema()](buildschema-aggfunction.md)|返回允许 `dynamic` 输入的所有值的最小架构|
|[count()](count-aggfunction.md)|返回组的计数|
|[countif()](countif-aggfunction.md)|返回具有组谓词的计数|
|[dcount()](dcount-aggfunction.md)|返回组元素的近似非重复计数|
|[dcountif()](dcountif-aggfunction.md)|返回组元素的近似非重复计数（带谓词）|
|[make_bag()](make-bag-aggfunction.md)|返回一个在组内包含动态值的属性包|
|[make_bag_if()](make-bag-if-aggfunction.md)|返回一个在组内包含动态值的属性包（带谓词）|
|[make_list()](makelist-aggfunction.md)|返回组中所有值的列表|
|[make_list_if()](makelistif-aggfunction.md)|返回组中所有值的列表（带谓词）|
|[make_list_with_nulls()](make-list-with-nulls-aggfunction.md)|返回组中所有值（包括 null 值）的列表|
|[make_set()](makeset-aggfunction.md)|返回组中非重复值的集合|
|[make_set_if()](makesetif-aggfunction.md)|返回组中非重复值的集合（带谓词）|
|[max()](max-aggfunction.md)|返回组内的最大值|
|[maxif()](maxif-aggfunction.md)|返回组中的最大值（带谓词）|
|[min()](min-aggfunction.md)|返回组内的最小值|
|[minif()](minif-aggfunction.md)|返回组中的最小值（带谓词）|
|[percentiles()](percentiles-aggfunction.md)|返回组的百分位近似值|
|[percentiles_array()](percentiles-aggfunction.md)|返回组的百分位近似值|
|[percentilesw()](percentiles-aggfunction.md)|返回组的加权百分位近似值|
|[percentilesw_array()](percentiles-aggfunction.md)|返回组的加权百分位近似值|
|[stdev()](stdev-aggfunction.md)|返回整个组的标准偏差|
|[stdevif()](stdevif-aggfunction.md)|返回整个组的标准偏差（带谓词）|
|[sum()](sum-aggfunction.md)|返回组中元素的总和|
|[sumif()](sumif-aggfunction.md)|返回组中元素的总和（带谓词）|
|[variance()](variance-aggfunction.md)|返回整个组的方差|
|[varianceif()](varianceif-aggfunction.md)|返回整个组的方差（带谓词）|

## <a name="aggregates-default-values"></a>对默认值进行聚合

下表汇总了聚合的默认值：

运算符       |默认值                         
---------------|------------------------------------
 `count()`, `countif()`, `dcount()`, `dcountif()`         |   0                            
 `make_bag()`, `make_bag_if()`, `make_list()`, `make_list_if()`, `make_set()`, `make_set_if()` |    空的动态数组              ([])          
 所有其他          |   null                           

 对包含 null 值的实体使用这些聚合时，null 值会被忽略，并且不会参与计算（请参阅下面的示例）。

## <a name="examples"></a>示例

:::image type="content" source="images/summarizeoperator/summarize-price-by-supplier.png" alt-text="按水果和供应商汇总价格":::

## <a name="example"></a>示例

确定表中有 `ActivityType` 和 `CompletionStatus` 的哪些唯一组合。 没有聚合函数，只是有分组依据键。 输出将只显示这些结果的列：

```kusto
Activities | summarize by ActivityType, completionStatus
```

|`ActivityType`|`completionStatus`
|---|---
|`dancing`|`started`
|`singing`|`started`
|`dancing`|`abandoned`
|`singing`|`completed`

## <a name="example"></a>示例

查找 Activities 表中所有记录的最小和最大时间戳。 没有 group by 子句，因此输出中只有一行：

```kusto
Activities | summarize Min = min(Timestamp), Max = max(Timestamp)
```

|`Min`|`Max`
|---|---
|`1975-06-09 09:21:45` | `2015-12-24 23:45:00`

## <a name="example"></a>示例

为每个大陆创建一行，并显示发生活动的城市的计数。 由于“continent”的值很少，因此“by”子句中不需要使用任何分组函数：

```kusto
Activities | summarize cities=dcount(city) by continent
```

|`cities`|`continent`
|---:|---
|`4290`|`Asia`|
|`3267`|`Europe`|
|`2673`|`North America`|


## <a name="example"></a>示例

下面的示例将计算每个活动类型的直方图。 由于 `Duration` 有许多值，因此请使用 `bin` 将其值按 10 分钟的间隔分组：

```kusto
Activities | summarize count() by ActivityType, length=bin(Duration, 10m)
```

|`count_`|`ActivityType`|`length`
|---:|---|---
|`354`| `dancing` | `0:00:00.000`
|`23`|`singing` | `0:00:00.000`
|`2717`|`dancing`|`0:10:00.000`
|`341`|`singing`|`0:10:00.000`
|`725`|`dancing`|`0:20:00.000`
|`2876`|`singing`|`0:20:00.000`
|...

**对默认值进行聚合的示例**

当 `summarize` 运算符的输入至少有一个空的分组依据键时，其结果也将为空。

如果 `summarize` 运算符的输入没有空的分组依据键，则结果将是在 `summarize` 中使用的聚合的默认值：

```kusto
datatable(x:long)[]
| summarize any(x), arg_max(x, x), arg_min(x, x), avg(x), buildschema(todynamic(tostring(x))), max(x), min(x), percentile(x, 55), hll(x) ,stdev(x), sum(x), sumif(x, x > 0), tdigest(x), variance(x)
```

|any_x|max_x|max_x_x|min_x|min_x_x|avg_x|schema_x|max_x1|min_x1|percentile_x_55|hll_x|stdev_x|sum_x|sumif_x|tdigest_x|variance_x|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|||||||||||||||||

```kusto
datatable(x:long)[]
| summarize  count(x), countif(x > 0) , dcount(x), dcountif(x, x > 0)
```

|count_x|countif_|dcount_x|dcountif_x|
|---|---|---|---|
|0|0|0|0|

```kusto
datatable(x:long)[]
| summarize  make_set(x), make_list(x)
```

|set_x|list_x|
|---|---|
|[]|[]|

聚合平均值运算会对所有非 null 值求和，只计算参与计算的那些值（不会将 null 值考虑在内）。

```kusto
range x from 1 to 2 step 1
| extend y = iff(x == 1, real(null), real(5))
| summarize sum(y), avg(y)
```

|sum_y|avg_y|
|---|---|
|5|5|

常规计数会将 null 计在内： 

```kusto
range x from 1 to 2 step 1
| extend y = iff(x == 1, real(null), real(5))
| summarize count(y)
```

|count_y|
|---|
|2|

```kusto
range x from 1 to 2 step 1
| extend y = iff(x == 1, real(null), real(5))
| summarize make_set(y), make_set(y)
```

|set_y|set_y1|
|---|---|
|[5.0]|[5.0]|
