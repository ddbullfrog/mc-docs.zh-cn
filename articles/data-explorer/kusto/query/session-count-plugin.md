---
title: session_count 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 session_count 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 02cbb26518680ea5a7cea732618e6309260870f0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105987"
---
# <a name="session_count-plugin"></a>session_count 插件

针对某个时间线基于 ID 列计算会话计数。

```kusto
T | evaluate session_count(id, datetime_column, startofday(ago(30d)), startofday(now()), 1min, 30min, dim1, dim2, dim3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `session_count(`*IdColumn*`,` *TimelineColumn*`,` *Start*`,` *End*`,` *Bin*`,` *LookBackWindow* [`,` *dim1*`,` *dim2*`,` ...]`)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* IdColumn：列的名称，其 ID 值表示用户活动。 
* TimelineColumn：表示时间线的列的名称。
* *开始* ：带有分析开始时段值的标量。
* End：带有分析结束时段值的标量。
* *Bin* ：会话分析步骤时间段的标量常数值。
* *LookBackWindow* ：表示会话回看期的标量常数值。 如果 `IdColumn` 中的 ID 在 `LookBackWindow` 之内的时间窗口中出现，则该会话被视为现有会话。 如果该 ID 未出现，则该会话被视为新会话。
* *dim1* , *dim2* , ...：（可选）维度列的列表，用于切分会话计数计算。

## <a name="returns"></a>返回

返回一个表，该表包含每个时间线期间的以及每个现有维度组合的会话计数值。

输出表架构如下：

|*TimelineColumn*|dim1|..|dim_n|count_sessions|
|---|---|---|---|---|--|--|--|--|--|--|
|类型：自 TimelineColumn 起|..|..|..|long|


## <a name="examples"></a>示例

对于此示例，数据是确定性的，并且我们使用具有两列的表：
- 时间线：一个 1 至 10,000 之间的顺序号
- ID：1 至 50 之间的用户 ID

如果 `Id` 是 `Timeline` 的除数（时间线 % Id == 0），则该 ID 会出现在特定的 `Timeline` 时段。

`Id==1` 的事件将出现在任何 `Timeline` 时段，`Id==2` 的事件将每隔一个 `Timeline` 时段出现，依此类推。

下面是很少的 20 行数据：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let _data = range Timeline from 1 to 10000 step 1
| extend __key = 1
| join kind=inner (range Id from 1 to 50 step 1 | extend __key=1) on __key
| where Timeline % Id == 0
| project Timeline, Id;
// Look on few lines of the data
_data
| order by Timeline asc, Id asc
| limit 20
```

|时间线|ID|
|---|---|
|1|1|
|2|1|
|2|2|
|3|1|
|3|3|
|4|1|
|4|2|
|4|4|
|5|1|
|5|5|
|6|1|
|6|2|
|6|3|
|6|6|
|7|1|
|7|7|
|8|1|
|8|2|
|8|4|
|8|8|

我们换一种说法来定义会话：只要用户 (`Id`) 在 100 个时段的时间范围内出现至少一次，而会话回看窗口是 41 个时段，则会话被视为活跃。

下一个查询根据上述定义显示活跃会话的计数。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let _data = range Timeline from 1 to 9999 step 1
| extend __key = 1
| join kind=inner (range Id from 1 to 50 step 1 | extend __key=1) on __key
| where Timeline % Id == 0
| project Timeline, Id;
// End of data definition
_data
| evaluate session_count(Id, Timeline, 1, 10000, 100, 41)
| render linechart 
```

:::image type="content" source="images/session-count-plugin/example-session-count.png" alt-text="会话计数示例" border="false":::
