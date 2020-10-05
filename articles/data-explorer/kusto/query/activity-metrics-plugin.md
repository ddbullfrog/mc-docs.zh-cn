---
title: activity_metrics 插件 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 activity_metrics 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: e833a992164b43b555276913fd1c22cd31083abe
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146605"
---
# <a name="activity_metrics-plugin"></a>activity_metrics 插件

根据当前阶段窗口与前一阶段窗口计算有用的活动指标（不同的计数值、新值的不同计数、保留率和流失率）（与 [activity_counts_metrics 插件](activity-counts-metrics-plugin.md)不同，在该插件中，每个时间窗口都与所有以前的时间窗口进行比较）。

```kusto
T | evaluate activity_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `activity_metrics(`*IdColumn*`,` *TimelineColumn*`,` [*Start*`,` *End*`,`] *Window* [`,` *dim1*`,` *dim2*`,` ...]`)`

## <a name="arguments"></a>参数

* *T*：输入表格表达式。
* *IdColumn*：列的名称，其 ID 值表示用户活动。 
* TimelineColumn：表示时间线的列的名称。
* *Start*：（可选）带有分析开始时段值的标量。
* *End*：（可选）带有分析结束时段值的标量。
* Window：带有分析窗口时段值的标量。 既可以是数字/日期/时间/时间戳值，也可以是 `week`/`month`/`year` 中的一个字符串（在这种情况下，这些时段分别为 [startofweek](startofweekfunction.md)/[startofmonth](startofmonthfunction.md)/[startofyear](startofyearfunction.md)）。 
* dim1, dim2, ... ：（可选）维度列的列表，用于切分活动指标计算。

## <a name="returns"></a>返回

返回一个表，该表包含每个时间线时段和每个现有维度组合的不同计数值、新值的不同计数、保留率和流失率。

输出表架构如下：

|TimelineColumn|dcount_values|dcount_newvalues|retention_rate|churn_rate|dim1|..|dim_n|
|---|---|---|---|---|--|--|--|--|--|--|
|类型：自 TimelineColumn 起|long|long|Double|Double|..|..|..|

**说明**

保留率定义

一段时间内 `Retention Rate` 的计算方法为：

> 在此期间返回的客户数  
> /（除以）  
> 此期间开始时的客户数  

其中的 `# of customers returned during the period` 定义为：

> 此期间结束时的客户数  
> \-（减）  
> 此期间获取的新客户数  

`Retention Rate` 可以在从 0.0 到 1.0 的范围内变化  
分数越高表示返回用户的数量越大。


流失率定义

一段时间内 `Churn Rate` 的计算方法为：
    
> 此期间流失的客户数  
> /（除以）  
> 此期间开始时的客户数  

其中的 `# of customer lost in the period` 定义为：

> 此期间开始时的客户数  
> \-（减）  
> 此期间结束时的客户数  

`Churn Rate` 可以在从 0.0 到 1.0 的范围内变化。分数越高，表示不返回到服务的用户数量越多。

流失率与保留率

派生自 `Churn Rate` 和 `Retention Rate` 的定义，以下公式始终成立：

> [`Retention Rate`] = 100.0% - [`Churn Rate`]


## <a name="examples"></a>示例

### <a name="weekly-retention-rate-and-churn-rate"></a>每周保留率和流失率

下一个查询计算每周保留率和流失率。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-02);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// 
| evaluate activity_metrics(['id'], _day, _start, _end, 7d)
| project _day, retention_rate, churn_rate
| render timechart 
```

|_day|retention_rate|churn_rate|
|---|---|---|
|2017-01-02 00:00:00.0000000|NaN|NaN|
|2017-01-09 00:00:00.0000000|0.179910044977511|0.820089955022489|
|2017-01-16 00:00:00.0000000|0.744374437443744|0.255625562556256|
|2017-01-23 00:00:00.0000000|0.612096774193548|0.387903225806452|
|2017-01-30 00:00:00.0000000|0.681141439205955|0.318858560794045|
|2017-02-06 00:00:00.0000000|0.278145695364238|0.721854304635762|
|2017-02-13 00:00:00.0000000|0.223172628304821|0.776827371695179|
|2017-02-20 00:00:00.0000000|0.38|0.62|
|2017-02-27 00:00:00.0000000|0.295519001701645|0.704480998298355|
|2017-03-06 00:00:00.0000000|0.280387770320656|0.719612229679344|
|2017-03-13 00:00:00.0000000|0.360628154795289|0.639371845204711|
|2017-03-20 00:00:00.0000000|0.288008028098344|0.711991971901656|
|2017-03-27 00:00:00.0000000|0.306134969325153|0.693865030674847|
|2017-04-03 00:00:00.0000000|0.356866537717602|0.643133462282398|
|2017-04-10 00:00:00.0000000|0.495098039215686|0.504901960784314|
|2017-04-17 00:00:00.0000000|0.198296836982968|0.801703163017032|
|2017-04-24 00:00:00.0000000|0.0618811881188119|0.938118811881188|
|2017-05-01 00:00:00.0000000|0.204657727593507|0.795342272406493|
|2017-05-08 00:00:00.0000000|0.517391304347826|0.482608695652174|
|2017-05-15 00:00:00.0000000|0.143667296786389|0.856332703213611|
|2017-05-22 00:00:00.0000000|0.199122325836533|0.800877674163467|
|2017-05-29 00:00:00.0000000|0.063468992248062|0.936531007751938|

:::image type="content" source="images/activity-metrics-plugin/activity-metrics-churn-and-retention.png" border="false" alt-text="活动指标改动和保留":::

### <a name="distinct-values-and-distinct-new-values"></a>不同的值和不同的“新”值 

下一个查询计算每周的不同值和“新”值（上一个时间窗口中没有出现的 ID）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-02);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// 
| evaluate activity_metrics(['id'], _day, _start, _end, 7d)
| project _day, dcount_values, dcount_newvalues
| render timechart 
```

|_day|dcount_values|dcount_newvalues|
|---|---|---|
|2017-01-02 00:00:00.0000000|630|630|
|2017-01-09 00:00:00.0000000|738|575|
|2017-01-16 00:00:00.0000000|1187|841|
|2017-01-23 00:00:00.0000000|1092|465|
|2017-01-30 00:00:00.0000000|1261|647|
|2017-02-06 00:00:00.0000000|1744|1043|
|2017-02-13 00:00:00.0000000|1563|432|
|2017-02-20 00:00:00.0000000|1406|818|
|2017-02-27 00:00:00.0000000|1956|1429|
|2017-03-06 00:00:00.0000000|1593|848|
|2017-03-13 00:00:00.0000000|1801|1423|
|2017-03-20 00:00:00.0000000|1710|1017|
|2017-03-27 00:00:00.0000000|1796|1516|
|2017-04-03 00:00:00.0000000|1381|1008|
|2017-04-10 00:00:00.0000000|1756|1162|
|2017-04-17 00:00:00.0000000|1831|1409|
|2017-04-24 00:00:00.0000000|1823|1164|
|2017-05-01 00:00:00.0000000|1811|1353|
|2017-05-08 00:00:00.0000000|1691|1246|
|2017-05-15 00:00:00.0000000|1812|1608|
|2017-05-22 00:00:00.0000000|1740|1017|
|2017-05-29 00:00:00.0000000|960|756|

:::image type="content" source="images/activity-metrics-plugin/activity-metrics-dcount-and-dcount-newvalues.png" border="false" alt-text="活动指标改动和保留":::
