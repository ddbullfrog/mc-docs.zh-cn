---
title: active_users_count 插件 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 active_users_count 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 595444076f5626ea01d16f04312c956a48097caa
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105110"
---
# <a name="active_users_count-plugin"></a>active_users_count 插件

计算值的非重复计数，其中每个值都至少已在回看期内最低数量的时间段中出现。

仅适用于计算“粉丝”的非重复计数，而不包括“非粉丝”的出现。 仅当用户在回看期中处于活跃状态时，该用户才会被计为“粉丝”。 回看期仅用于确定用户是否被视为 `active`（“粉丝”）。 聚合本身不包括来自回看窗口的用户。 相比之下，[sliding_window_counts](sliding-window-counts-plugin.md) 聚合针对回看期的滑动窗口执行。

```kusto
T | evaluate active_users_count(id, datetime_column, startofday(ago(30d)), startofday(now()), 7d, 1d, 2, 7d, dim1, dim2, dim3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `active_users_count(`*IdColumn*`,` *TimelineColumn*`,` *Start*`,` *End*`,` *LookbackWindow*`,` *Period*`,` *ActivePeriodsCount*`,` *Bin* `,` [ *dim1*`,` *dim2*`,` ...]`)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* *IdColumn* ：列的名称，其 ID 值表示用户活跃度。 
* *TimelineColumn* ：表示时间线的列的名称。
* *Start* ：（可选）带有分析开始时间段值的标量。
* *End* ：（可选）带有分析结束时间段值的标量。
* *LookbackWindow* ：滑动时间窗口，限定检查用户出现情况的时间段。 回看期开始于（[当前出现] - [回看窗口]），结束于（[当前出现]）。 
* *Period* ：要计为单次出现的标量常数时间跨度（如果用户在此时间跨度的至少非重复的 ActivePeriodsCount 个时间段中出现，该用户将会被计为活跃）。
* *ActivePeriodsCount* ：非重复的活跃时间段的最小数目，用于确定用户是否处于活跃状态。 活跃用户是指那些至少在（等于或大于）最低数量的活跃时间段中出现的用户。
* *Bin* ：分析步骤时间段的标量常数值。 可以是数字/日期/时间/时间戳值，也可以是 `week`/`month`/`year` 字符串。 所有时间段都将会是相应的 [startofweek](startofweekfunction.md)/[startofmonth](startofmonthfunction.md)/[startofyear](startofyearfunction.md) 函数。
* *dim1* , *dim2* , ...：（可选）维度列的列表，用于切分活跃度指标计算。

## <a name="returns"></a>返回

返回一个表，该表包含 ID 的非重复计数值，这些 ID 已在以下时间段中的 ActivePeriodCounts 个时间段中出现：回看期、每个时间线时间段，以及每个现有的维度组合。

输出表架构如下：

|*TimelineColumn*|dim1|..|dim_n|dcount_values|
|---|---|---|---|---|
|类型：自 TimelineColumn 起|..|..|..|long|


## <a name="examples"></a>示例

计算每周的非重复用户数，这些用户在前八天的时间段内有至少不同三天都出现过。 分析的时间段：2018 年 7 月。

```kusto
let Start = datetime(2018-07-01);
let End = datetime(2018-07-31);
let LookbackWindow = 8d;
let Period = 1d;
let ActivePeriods = 3;
let Bin = 7d; 
let T =  datatable(User:string, Timestamp:datetime)
[
    "B",      datetime(2018-06-29),
    "B",      datetime(2018-06-30),
    "A",      datetime(2018-07-02),
    "B",      datetime(2018-07-04),
    "B",      datetime(2018-07-08),
    "A",      datetime(2018-07-10),
    "A",      datetime(2018-07-14),
    "A",      datetime(2018-07-17),
    "A",      datetime(2018-07-20),
    "B",      datetime(2018-07-24)
]; 
T | evaluate active_users_count(User, Timestamp, Start, End, LookbackWindow, Period, ActivePeriods, Bin)



```

|Timestamp|`dcount`|
|---|---|
|2018-07-01 00:00:00.0000000|1|
|2018-07-15 00:00:00.0000000|1|

如果以下两个条件都满足，则用户会被视为处于活跃状态： 
* 在至少三个不同日期（Period = 1d，ActivePeriods = 3）中发现该用户。
* 在 8 天的回看窗口（早于并包括当前这次出现）中发现该用户。

在下图中，符合此活跃状态条件的出现只有以下实例：用户 A 在 7 月 20 日出现以及用户 B 在 7 月 4 日出现（请参阅上述插件结果）。 用户 B 的出现包括在 7 月 4 日的回看窗口中，但不包括在 6 月 29 日至 30 日的 Start-End 时间范围内。 

:::image type="content" source="images/queries/active-users-count.png" alt-text="活跃用户计数示例":::

