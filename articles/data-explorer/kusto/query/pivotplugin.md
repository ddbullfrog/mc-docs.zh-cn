---
title: pivot 插件 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 pivot 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 53e13724a336c0240f98b87800b32ece3730be2d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103833"
---
# <a name="pivot-plugin"></a>pivot 插件

通过将输入表的某一列中的唯一值转换为输出表中的多个列来旋转表，并在必要时对最终输出中所需的任何其余列值执行聚合。

```kusto
T | evaluate pivot(PivotColumn)
```

> [!NOTE]
> `pivot` 插件的输出架构基于数据，因此，查询可能会为任意两个运行生成不同的架构。 这也意味着，引用未包装列的查询随时可能会“中断”。 由于此原因，不建议将此插件用于自动化作业。

## <a name="syntax"></a>语法

`T | evaluate pivot(`*pivotColumn*`[, `*aggregationFunction*`] [,`*column1* `[,`*column2* ... `]])`

## <a name="arguments"></a>参数

* pivotColumn：要旋转的列。 此列中的每个唯一值将是输出表中的一个列。
* 聚合函数：（可选）将输入表中的多个行聚合到输出表中的单个行。 当前支持的函数：`min()`、`max()`、`any()`、`sum()`、`dcount()`、`avg()`、`stdev()`、`variance()`、`make_list()`、`make_bag()`、`make_set()`、`count()`（默认值为 `count()`）。
* column1、column2……：（可选）列名称。 输出表将为每个指定列包含一个额外的列。 默认值：除已透视列和聚合列以外的所有列。

## <a name="returns"></a>返回

Pivot 返回包含指定列（ *column1* 、 *column2* ……）及透视列的所有唯一值的旋转表。 已透视列的每个单元格都将包含聚合函数计算。

## <a name="examples"></a>示例

### <a name="pivot-by-a-column"></a>按列透视

对于以“AL”开头的每个 EventType 和 State，计算此州内该类型的事件数。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| project State, EventType 
| where State startswith "AL" 
| where EventType has "Wind" 
| evaluate pivot(State)
```

|EventType|ALABAMA|ALASKA|
|---|---|---|
|雷雨大风|352|1|
|High Wind|0|95|
|Extreme Cold/Wind Chill|0|10|
|Strong Wind|22|0|


### <a name="pivot-by-a-column-with-aggregation-function"></a>使用聚合函数按列进行透视

对于每个以“AR”开头的 EventType 和 State，显示直接死亡人数的总数。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where State startswith "AR" 
| project State, EventType, DeathsDirect 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect))
```

|EventType|ARKANSAS|ARIZONA|
|---|---|---|
|暴雨|1|0|
|雷雨大风|1|0|
|Lightning|0|1|
|山洪|0|6|
|Strong Wind|1|0|
|热|3|0|


### <a name="pivot-by-a-column-with-aggregation-function-and-a-single-additional-column"></a>按包含聚合函数的列和单个附加列进行透视

结果与上一示例相同。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where State startswith "AR" 
| project State, EventType, DeathsDirect 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect), EventType)
```

|EventType|ARKANSAS|ARIZONA|
|---|---|---|
|暴雨|1|0|
|雷雨大风|1|0|
|Lightning|0|1|
|山洪|0|6|
|Strong Wind|1|0|
|热|3|0|


### <a name="specify-the-pivoted-column-aggregation-function-and-multiple-additional-columns"></a>指定已透视列、聚合函数和多个附加列

对于每个事件类型、源和州，计算直接死亡人数之和。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where State startswith "AR" 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect), EventType, Source)
```

|EventType|Source|ARKANSAS|ARIZONA|
|---|---|---|---|
|暴雨|灾害管理|1|0|
|雷雨大风|灾害管理|1|0|
|Lightning|报纸|0|1|
|山洪|专业观测员|0|2|
|山洪|Broadcast Media|0|3|
|山洪|报纸|0|1|
|Strong Wind|执法机构|1|0|
|热|报纸|3|0|
