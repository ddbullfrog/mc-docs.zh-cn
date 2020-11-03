---
title: sliding_window_counts 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 sliding_window_counts 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 057ed19b2be6ceac1b641c146b9d20c6fe448cde
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106114"
---
# <a name="sliding_window_counts-plugin"></a>sliding_window_counts 插件

使用[此处](samples.md#perform-aggregations-over-a-sliding-window)所述的技术，计算回溯期间滑动窗口中的值的计数和非重复计数。

例如，对于每一天，计算上周的用户计数和非重复计数 。 

```kusto
T | evaluate sliding_window_counts(id, datetime_column, startofday(ago(30d)), startofday(now()), 7d, 1d, dim1, dim2, dim3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `sliding_window_counts(`*IdColumn*`,` *TimelineColumn*`,` *Start*`,` *End*`,` *LookbackWindow*`,` *Bin* `,` [ *dim1*`,` *dim2*`,` ...]`)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* *IdColumn* ：列的名称，其 ID 值表示用户活动。 
* TimelineColumn：表示时间线的列的名称。
* *开始* ：带有分析开始时段值的标量。
* End：带有分析结束时段值的标量。
* *LookbackWindow* ：回溯期间的标量常数值（例如，对于过去 7 天的 `dcount` 用户：LookbackWindow = 7d）
* *Bin* ：分析步骤时间段的标量常数值。 此值可以是数字/日期时间/时间戳值。 如果该值是格式为 `week`/`month`/`year` 的字符串，则所有期间都将为 [startofweek](startofweekfunction.md)/[startofmonth](startofmonthfunction.md)/[startofyear](startofyearfunction.md)。 
* dim1, dim2, ... ：（可选）维度列的列表，用于切分活动指标计算。

## <a name="returns"></a>返回

返回一个表，该表包含回溯期间每个时间线时段（根据 bin）和每个现有维度组合的 ID 的计数和非重复计数值。

输出表架构如下：

|TimelineColumn|`dim1`|..|`dim_n`|`count`|`dcount`|
|---|---|---|---|---|---|
|类型：自 TimelineColumn 起|..|..|..|long|long|


## <a name="examples"></a>示例

计算过去一周内每天分析时段的用户计数和 `dcounts`。 

```kusto
let start = datetime(2017 - 08 - 01);
let end = datetime(2017 - 08 - 07); 
let lookbackWindow = 3d;  
let bin = 1d;
let T = datatable(UserId:string, Timestamp:datetime)
[
'Bob',      datetime(2017 - 08 - 01), 
'David',    datetime(2017 - 08 - 01), 
'David',    datetime(2017 - 08 - 01), 
'John',     datetime(2017 - 08 - 01), 
'Bob',      datetime(2017 - 08 - 01), 
'Ananda',   datetime(2017 - 08 - 02),  
'Atul',     datetime(2017 - 08 - 02), 
'John',     datetime(2017 - 08 - 02), 
'Ananda',   datetime(2017 - 08 - 03), 
'Atul',     datetime(2017 - 08 - 03), 
'Atul',     datetime(2017 - 08 - 03), 
'John',     datetime(2017 - 08 - 03), 
'Bob',      datetime(2017 - 08 - 03), 
'Betsy',    datetime(2017 - 08 - 04), 
'Bob',      datetime(2017 - 08 - 05), 
];
T | evaluate sliding_window_counts(UserId, Timestamp, start, end, lookbackWindow, bin)


```

|Timestamp|计数|`dcount`|
|---|---|---|
|2017-08-01 00:00:00.0000000|5|3|
|2017-08-02 00:00:00.0000000|8|5|
|2017-08-03 00:00:00.0000000|13|5|
|2017-08-04 00:00:00.0000000|9|5|
|2017-08-05 00:00:00.0000000|7|5|
|2017-08-06 00:00:00.0000000|2|2|
|2017-08-07 00:00:00.0000000|1|1|