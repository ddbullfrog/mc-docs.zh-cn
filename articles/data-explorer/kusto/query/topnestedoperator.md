---
title: top-nested 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 top-nested 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: b75fd54d6cff671839010d73f2cd1fe1d1947eb8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105310"
---
# <a name="top-nested-operator"></a>top-nested 运算符

生成分层聚合和前位值选择，其中每个级别都是前一级别的优化。

```kusto
T | top-nested 3 of Location with others="Others" by sum(MachinesNumber), top-nested 4 of bin(Timestamp,5m) by sum(MachinesNumber)
```

`top-nested` 运算符接受表格数据作为输入，以及一个或多个聚合子句。
第一个聚合子句（最左侧）根据针对这些记录的某个表达式的惟一值，将输入记录细分为多个分区。 然后，子句会持有一定数量的记录，这可以最大化或最小化针对记录的这个表达式。 然后，下一个聚合子句以嵌套方式应用一个类似的函数。 以下每个子句都应用于前一个子句生成的分区。 将持续为所有聚合子句执行此过程。

例如，`top-nested` 运算符可用于解答以下问题：“对于一个包含销售数据（如国家/地区、销售人员和销售数量）的表：按销售额排名前五位的国家/地区分别有哪些？ 每个国家/地区排名前三的销售人员是哪几位？”

## <a name="syntax"></a>语法

*T* `|` `top-nested` *TopNestedClause2* [`,` *TopNestedClause2* ...]

其中 TopNestedClause 具有以下语法：

[ *N* ] `of` [ *`ExprName`* `=`] *`Expr`* [`with` `others` `=` *`ConstExpr`* ] `by` [ *`AggName`* `=`] *`Aggregation`* [`asc` | `desc`]

## <a name="arguments"></a>参数

对于每个 TopNestedClause：

* *`N`* ：`long` 类型的文本，指示要为此层次结构级别返回多少前位值。
  如果省略，将返回所有非重复值。

* *`ExprName`* ：如果指定，则设置与 `Expr` 值相对应的输出列的名称。

* *`Expr`* ：针对输入记录的表达式，指示要为此层次结构级别返回的值。
  通常它是表格式输入 (T) 的列引用，或针对此类列的某些计算（如 `bin()`）的列引用。

* *`ConstExpr`* ：如果指定，对于每个层次结构级别，将添加 1 个记录，其值是所有未“使其成为前位值”的记录的聚合值。

* *`AggName`* ：如果指定此标识符，则该标识符将在输出中为 Aggregation 的值设置列名称。

* *`Aggregation`* ：一个数值表达式，指示要应用于具有相同值 `Expr` 的所有记录的聚合。 这个聚合的值决定了得到的记录中哪些排在“前位”。
  
  支持以下聚合函数：
   * [sum()](sum-aggfunction.md)、
   * [count()](count-aggfunction.md)、
   * [max()](max-aggfunction.md)、
   * [min()](min-aggfunction.md)、
   * [dcount()](dcountif-aggfunction.md)、
   * [avg()](avg-aggfunction.md)、
   * [percentile()](percentiles-aggfunction.md) 和
   * [percentilew()](percentiles-aggfunction.md)。 还支持聚合的任何代数组合。

* `asc` 或 `desc`（默认）可能会控制实际从聚合值范围“底部”还是“顶部”进行选择。

## <a name="returns"></a>返回

此运算符返回一个表，其中每个聚合子句有两列：

* 其中一列包含子句的 `Expr` 计算的非重复值（列名为 ExprName，如果指定了的话） 

* 其中一列保存 Aggregation 计算的结果（列名为 AggregationName，如果指定了的话） 

## <a name="notes"></a>说明

不输出未指定为 `Expr` 值的输入列。
若要获取特定级别的所有值，请添加一个聚合计数：

* 省略 N 的值
* 使用列名称作为 `Expr` 的值
* 使用 `Ignore=max(1)` 作为该聚合，然后忽略（或 project-away）`Ignore` 列。

记录的数量可能随着聚合子句 ((N1+1) \* (N2+1) \* ...) 的数量呈指数增长。如果未指定 N 限制，记录会增长更快。 使用时应考虑到此运算符可能会消耗大量的资源。

如果聚合的分布相当不均匀，请限制返回的非重复值的数量（通过使用 N），并使用 `with others=` ConstExpr 选项来获得所有其他情况的“权重”指示 。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation by sum(BeginLat)
```

|状态|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|KANSAS|87771.2355000001|执法机构|18744.823|FT SCOTT|264.858|
|KANSAS|87771.2355000001|公用|22855.6206|BUCKLIN|488.2457|
|KANSAS|87771.2355000001|专业观测员|21279.7083|SHARON SPGS|388.7404|
|德克萨斯|123400.5101|公用|13650.9079|AMARILLO|246.2598|
|德克萨斯|123400.5101|执法机构|37228.5966|PERRYTON|289.3178|
|德克萨斯|123400.5101|专业观测员|13997.7124|CLAUDE|421.44|

Use the option 'with others':

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State with others = "All Other States" by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation with others = "All Other End Locations" by  sum(BeginLat)


```

|状态|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|KANSAS|87771.2355000001|执法机构|18744.823|FT SCOTT|264.858|
|KANSAS|87771.2355000001|公用|22855.6206|BUCKLIN|488.2457|
|KANSAS|87771.2355000001|专业观测员|21279.7083|SHARON SPGS|388.7404|
|德克萨斯|123400.5101|公用|13650.9079|AMARILLO|246.2598|
|德克萨斯|123400.5101|执法机构|37228.5966|PERRYTON|289.3178|
|德克萨斯|123400.5101|专业观测员|13997.7124|CLAUDE|421.44|
|KANSAS|87771.2355000001|执法机构|18744.823|所有其他结束位置|18479.965|
|KANSAS|87771.2355000001|公用|22855.6206|所有其他结束位置|22367.3749|
|KANSAS|87771.2355000001|专业观测员|21279.7083|所有其他结束位置|20890.9679|
|德克萨斯|123400.5101|公用|13650.9079|所有其他结束位置|13404.6481|
|德克萨斯|123400.5101|执法机构|37228.5966|所有其他结束位置|36939.2788|
|德克萨斯|123400.5101|专业观测员|13997.7124|所有其他结束位置|13576.2724|
|KANSAS|87771.2355000001|||所有其他结束位置|24891.0836|
|德克萨斯|123400.5101|||所有其他结束位置|58523.2932000001|
|所有其他州|1149279.5923|||所有其他结束位置|1149279.5923|

以下查询针对上例中使用的第一个级别显示了相同结果。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
 StormEvents
 | where State !in ('TEXAS', 'KANSAS')
 | summarize sum(BeginLat)
```

|sum_BeginLat|
|---|
|1149279.5923|


请求在顶部嵌套结果中设置另一列 (EventType)。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 1 of EndLocation by sum(BeginLat), top-nested of EventType  by tmp = max(1)
| project-away tmp
```

|状态|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|EventType|
|---|---|---|---|---|---|---|
|KANSAS|87771.2355000001|专业观测员|21279.7083|SHARON SPGS|388.7404|雷雨大风|
|KANSAS|87771.2355000001|专业观测员|21279.7083|SHARON SPGS|388.7404|冰雹|
|KANSAS|87771.2355000001|专业观测员|21279.7083|SHARON SPGS|388.7404|龙卷风|
|KANSAS|87771.2355000001|公用|22855.6206|BUCKLIN|488.2457|冰雹|
|KANSAS|87771.2355000001|公用|22855.6206|BUCKLIN|488.2457|雷雨大风|
|KANSAS|87771.2355000001|公用|22855.6206|BUCKLIN|488.2457|洪水|
|德克萨斯|123400.5101|专业观测员|13997.7124|CLAUDE|421.44|冰雹|
|德克萨斯|123400.5101|执法机构|37228.5966|PERRYTON|289.3178|冰雹|
|德克萨斯|123400.5101|执法机构|37228.5966|PERRYTON|289.3178|洪水|
|德克萨斯|123400.5101|执法机构|37228.5966|PERRYTON|289.3178|山洪|

为此级别（每个组）中的每个值指定索引排序顺序，以按最后一个嵌套级别（在本例中按 EndLocation）对结果排序：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State  by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 4 of EndLocation by  sum(BeginLat)
| order by State , Source, aggregated_EndLocation
| summarize EndLocations = make_list(EndLocation, 10000) , endLocationSums = make_list(aggregated_EndLocation, 10000) by State, Source
| extend indicies = range(0, array_length(EndLocations) - 1, 1)
| mv-expand EndLocations, endLocationSums, indicies
```

|状态|Source|EndLocations|endLocationSums|索引|
|---|---|---|---|---|
|德克萨斯|专业观测员|CLAUDE|421.44|0|
|德克萨斯|专业观测员|AMARILLO|316.8892|1|
|德克萨斯|专业观测员|DALHART|252.6186|2|
|德克萨斯|专业观测员|PERRYTON|216.7826|3|
|德克萨斯|执法机构|PERRYTON|289.3178|0|
|德克萨斯|执法机构|LEAKEY|267.9825|1|
|德克萨斯|执法机构|BRACKETTVILLE|264.3483|2|
|德克萨斯|执法机构|GILMER|261.9068|3|
|KANSAS|专业观测员|SHARON SPGS|388.7404|0|
|KANSAS|专业观测员|ATWOOD|358.6136|1|
|KANSAS|专业观测员|LENORA|317.0718|2|
|KANSAS|专业观测员|SCOTT CITY|307.84|3|
|KANSAS|公用|BUCKLIN|488.2457|0|
|KANSAS|公用|ASHLAND|446.4218|1|
|KANSAS|公用|PROTECTION|446.11|2|
|KANSAS|公用|MEADE STATE PARK|371.1|3|
