---
title: funnel_sequence_completion 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 funnel_sequence_completion 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/16/2020
ms.date: 10/29/2020
ms.openlocfilehash: fd7bb891a43d6af9fe2e798c291947d24e080ec9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104301"
---
# <a name="funnel_sequence_completion-plugin"></a>funnel_sequence_completion 插件

计算已完成的序列步骤（在起对比作用的不同时间段内）的漏斗。

```kusto
T | evaluate funnel_sequence_completion(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, state_column, dynamic(['S1', 'S2', 'S3']), dynamic([10m, 30min, 1h]))
```

## <a name="syntax"></a>语法

*T* `| evaluate` `funnel_sequence_completion(`*IdColumn*`,` *TimelineColumn*`,` *Start*`,` *End*`,` *Step*`,` *StateColumn*`,` *Sequence*`,` *MaxSequenceStepWindows*`)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* *IdColum* ：列引用，必须出现在源表达式中。
* *TimelineColumn* ：表示时间线的列引用，必须出现在源表达式中。
* *Start* ：分析开始时间段的标量常数值。
* *End* ：分析结束时间段的标量常数值。
* *Step* ：分析步骤时间段（箱）的标量常数值。
* *StateColumn* ：表示状态的列引用，必须出现在源表达式中。
* *Sequence* ：常数动态数组，包含序列值（在 `StateColumn` 中查找值）。
* *MaxSequenceStepWindows* ：标量常数动态数组，包含序列中第一个和最后一个顺序步骤之间所允许的时间跨度最大值。 该数组中每个窗口（时间段）都会生成漏斗分析结果。

## <a name="returns"></a>返回

返回一个表，用于为所分析的序列构造漏斗图：

* `TimelineColumn`：分析的时间窗口
* `StateColumn`：序列的状态。
* `Period`：可用于完成漏斗序列中步骤（从该序列中第一步度量）的最大时间段（窗口）。 MaxSequenceStepWindows 中的每个值都会生成与某个单独时间段相关的漏斗分析。 
* `dcount`：从第一个序列状态转换到 `StateColumn` 值的时间窗口中 `IdColumn` 的非重复计数。

## <a name="examples"></a>示例

### <a name="exploring-storm-events"></a>暴风雨活动探究 

以下查询检查序列（“总体”时间分别为 1 小时、4 小时和 1 天的 `Hail` -> `Tornado` -> `Thunderstorm Wind`）的完成漏斗。 

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let _start = datetime(2007-01-01);
let _end =  datetime(2008-01-01);
let _windowSize = 365d;
let _sequence = dynamic(['Hail', 'Tornado', 'Thunderstorm Wind']);
let _periods = dynamic([1h, 4h, 1d]);
StormEvents
| evaluate funnel_sequence_completion(EpisodeId, StartTime, _start, _end, _windowSize, EventType, _sequence, _periods) 
```

|`StartTime`|`EventType`|`Period`|`dcount`|
|---|---|---|---|
|2007-01-01 00:00:00.0000000|冰雹|01:00:00|2877|
|2007-01-01 00:00:00.0000000|龙卷风|01:00:00|208|
|2007-01-01 00:00:00.0000000|雷雨大风|01:00:00|87|
|2007-01-01 00:00:00.0000000|冰雹|04:00:00|2877|
|2007-01-01 00:00:00.0000000|龙卷风|04:00:00|231|
|2007-01-01 00:00:00.0000000|雷雨大风|04:00:00|141|
|2007-01-01 00:00:00.0000000|冰雹|1.00:00:00|2877|
|2007-01-01 00:00:00.0000000|龙卷风|1.00:00:00|244|
|2007-01-01 00:00:00.0000000|雷雨大风|1.00:00:00|155|

理解结果：  
结果为三个漏斗（所针对的时间段：1 小时、4 小时和 1 天）。 对于每个漏斗步骤，会显示多个非重复计数。 可以看出，给予完成整个 `Hail` -> `Tornado` -> `Thunderstorm Wind` 序列的时间越长，获得的 `dcount` 值就越高。 换句话说，到达漏斗步骤的序列发生的次数也就越多。
