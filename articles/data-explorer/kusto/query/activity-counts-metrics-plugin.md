---
title: activity_counts_metrics 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 activity_counts_metrics 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9448011c4596bc2eb52b48684a442b2636a5a146
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105102"
---
# <a name="activity_counts_metrics-plugin"></a>activity_counts_metrics 插件

计算每个时间窗口与所有先前的时间窗口进行比较/聚合的实用活动指标。 这些指标包括：总计数值、非重复计数值、新值的非重复计数和聚合非重复计数。 将此插件与 [activity_metrics 插件](activity-metrics-plugin.md)进行比较；在 activity_metrics 插件中，仅将每个时间窗口与其前一个时间窗口进行比较。

```kusto
T | evaluate activity_counts_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `activity_counts_metrics(`*IdColumn*`,` *TimelineColumn*`,` *Start*`,` *End*`,` *Window* [`,` *Cohort* ] [`,` *dim1*`,` *dim2*`,` ...] [`,` *Lookback* ] `)`

## <a name="arguments"></a>参数

* T：输入表格表达式。
* IdColumn：列的名称，其 ID 值表示用户活动。 
* TimelineColumn：表示时间线的列的名称。
* *开始* ：带有分析开始时段值的标量。
* End：带有分析结束时段值的标量。
* Window：带有分析窗口时段值的标量。 既可以是数字/日期时间/时间戳值，也可以是 `week`/`month`/`year` 中的一个字符串（在这种情况下，所有时段都将是 [startofweek](startofweekfunction.md)/[startofmonth](startofmonthfunction.md) 或 [startofyear](startofyearfunction.md)）。 
* dim1, dim2, ... ：（可选）维度列的列表，用于切分活动指标计算。

## <a name="returns"></a>返回

返回一个表，其中包括每个时间窗口的总计数值、非重复计数值、新值的非重复计数和聚合非重复计数。

输出表架构如下：

|`TimelineColumn`|`dim1`|...|`dim_n`|`count`|`dcount`|`new_dcount`|`aggregated_dcount`
|---|---|---|---|---|---|---|---|---|
|类型：自 `TimelineColumn` 起|..|..|..|long|long|long|long|long


* *`TimelineColumn`* ：时间窗口开始时间。
* *`count`* ：时间窗口和 dim 中的总记录数
* *`dcount`* ：时间窗口和 dim 中的非重复 ID 值
* *`new_dcount`* ：与之前的所有时间窗口相比，时间窗口和 dim 中的非重复 ID 值。 
* *`aggregated_dcount`* ：从第一个时间窗口到当前时间窗口（含），dim 的聚合非重复 ID 值总和。

## <a name="examples"></a>示例

### <a name="daily-activity-counts"></a>每日活动计数 

下一个查询为提供的输入表计算每日活动计数

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let start=datetime(2017-08-01);
let end=datetime(2017-08-04);
let window=1d;
let T = datatable(UserId:string, Timestamp:datetime)
[
'A', datetime(2017-08-01),
'D', datetime(2017-08-01), 
'J', datetime(2017-08-01),
'B', datetime(2017-08-01),
'C', datetime(2017-08-02),  
'T', datetime(2017-08-02),
'J', datetime(2017-08-02),
'H', datetime(2017-08-03),
'T', datetime(2017-08-03),
'T', datetime(2017-08-03),
'J', datetime(2017-08-03),
'B', datetime(2017-08-03),
'S', datetime(2017-08-03),
'S', datetime(2017-08-04),
];
 T 
 | evaluate activity_counts_metrics(UserId, Timestamp, start, end, window)
```

|`Timestamp`|`count`|`dcount`|`new_dcount`|`aggregated_dcount`|
|---|---|---|---|---|
|2017-08-01 00:00:00.0000000|4|4|4|4|
|2017-08-02 00:00:00.0000000|3|3|2|6|
|2017-08-03 00:00:00.0000000|6|5|2|8|
|2017-08-04 00:00:00.0000000|1|1|0|8|


