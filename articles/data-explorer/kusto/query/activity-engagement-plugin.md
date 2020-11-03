---
title: activity_engagement 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 activity_engagement 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9595bf008053742c6c4e01a79570c6932bb21d5a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105828"
---
# <a name="activity_engagement-plugin"></a>activity_engagement 插件

在滑动时间线窗口中，根据 ID 列计算活动参与比率。

activity_engagement 插件可用于计算 DAU/WAU/MAU（每日/每周/每月活动）。

```kusto
T | evaluate activity_engagement(id, datetime_column, 1d, 30d)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `activity_engagement(`*IdColumn*`,` *TimelineColumn*`,` [ *Start*`,` *End*`,`] *InnerActivityWindow*`,` *OuterActivityWindow* [`,` *dim1*`,` *dim2*`,` ...]`)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* *IdColumn* ：列的名称，其 ID 值表示用户活动。 
* TimelineColumn：表示时间线的列的名称。
* *Start* ：（可选）带有分析开始时段值的标量。
* *End* ：（可选）带有分析结束时段值的标量。
* *InnerActivityWindow* ：带有内部范围分析窗口时段值的标量。
* *OuterActivityWindow* ：带有外部范围分析窗口时段值的标量。
* dim1, dim2, ... ：（可选）维度列的列表，用于切分活动指标计算。

## <a name="returns"></a>返回

返回一个表，该表对于每个内部范围窗口时段和每个现有维度组合都有（内部范围窗口内 ID 值的不同计数、外部范围窗口内 ID 值的不同计数以及活动率）。

输出表架构如下：

|TimelineColumn|dcount_activities_inner|dcount_activities_outer|activity_ratio|dim1|..|dim_n|
|---|---|---|---|--|--|--|--|--|--|
|类型：自 TimelineColumn 起|long|long|Double|..|..|..|


## <a name="examples"></a>示例

### <a name="dauwau-calculation"></a>DAU/WAU 计算

下面的示例通过随机生成的数据计算 DAU/WAU（每日活动用户/每周活动用户比率）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-01-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Calculate DAU/WAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 7d)
| project _day, Dau_Wau=activity_ratio*100 
| render timechart 
```

:::image type="content" source="images/activity-engagement-plugin/activity-engagement-dau-wau.png" border="false" alt-text="活动参与 dau wau":::

### <a name="daumau-calculation"></a>DAU/MAU 计算

下面的示例通过随机生成的数据计算 DAU/WAU（每日活动用户/每周活动用户比率）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Calculate DAU/MAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 30d)
| project _day, Dau_Mau=activity_ratio*100 
| render timechart 
```

:::image type="content" source="images/activity-engagement-plugin/activity-engagement-dau-mau.png" border="false" alt-text="活动参与 dau wau":::

### <a name="daumau-calculation-with-additional-dimensions"></a>使用附加维度计算 DAU/MAU

下面的示例使用附加维度 (`mod3`) 通过随机生成的数据计算 DAU/WAU（每日活动用户/每周活动用户比率）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
| extend mod3 = strcat("mod3=", id % 3)
// Calculate DAU/MAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 30d, mod3)
| project _day, Dau_Mau=activity_ratio*100, mod3 
| render timechart 
```

:::image type="content" source="images/activity-engagement-plugin/activity-engagement-dau-mau-mod3.png" border="false" alt-text="活动参与 dau wau":::
