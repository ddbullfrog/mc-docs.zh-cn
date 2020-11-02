---
title: 在时间范围内进行联接 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的“在时间范围内进行联接”功能。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 57d02b1bf0c3f1568707faa5e345ad6bd6b1a65b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105597"
---
# <a name="time-window-join"></a>时间范围联接

我们经常需要基于某个高基数键（例如操作 ID 或会话 ID）在两个大型数据集之间进行联接，并通过对左右两侧的日期/时间列之间的“时间距离”添加限制来进一步限制需要与每个左侧 ($left) 记录进行匹配的右侧 ($right) 记录。

函数在联接中十分有用，例如在以下场景中：
* 基于某个高基数键（例如操作 ID 或会话 ID）在两个大型数据集之间进行联接。
* 通过对左右两侧的日期/时间列之间的“时间距离”添加限制，限制需要与每个左侧 ($left) 记录进行匹配的右侧 ($right) 记录。

上述操作不同于一般的 Kusto 联接操作，因为对于在左右两侧数据集之间匹配高基数键的 *`equi-join`* 部分，系统还可以应用距离函数并使用它大大加快联接速度。

> [!NOTE]
> 距离函数的行为与等式不同（也就是说，当 dist(x,y) 和 dist(y,z) 都为 true 时，它并不遵循 dist(x,z) 也为 true 的规则。）在内部，我们有时将其称为“对角联接”。

例如，如果你希望标识相对较小的时间范围内的事件序列，假设你有一个具有以下架构的表 `T`：

* `SessionId`：一个 `string` 类型的列，其中包含相关 ID。
* `EventType`：一个 `string` 类型的列，用于标识记录的事件类型。
* `Timestamp`：一个 `datetime` 类型的列，用于指示记录描述的事件是何时发生的。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let T = datatable(SessionId:string, EventType:string, Timestamp:datetime)
[
    '0', 'A', datetime(2017-10-01 00:00:00),
    '0', 'B', datetime(2017-10-01 00:01:00),
    '1', 'B', datetime(2017-10-01 00:02:00),
    '1', 'A', datetime(2017-10-01 00:03:00),
    '3', 'A', datetime(2017-10-01 00:04:00),
    '3', 'B', datetime(2017-10-01 00:10:00),
];
T
```

|SessionId|EventType|时间戳|
|---|---|---|
|0|A|2017-10-01 00:00:00.0000000|
|0|B|2017-10-01 00:01:00.0000000|
|1|B|2017-10-01 00:02:00.0000000|
|1|A|2017-10-01 00:03:00.0000000|
|3|A|2017-10-01 00:04:00.0000000|
|3|B|2017-10-01 00:10:00.0000000|


**问题陈述**

我们的查询应当回答以下问题：

   查找 `1min` 时间范围内在事件类型 `A` 后跟有事件类型 `B` 的所有会话 ID。

> [!NOTE]
> 在上面的示例数据中，这样的唯一会话 ID 是 `0`。

从语义上讲，以下查询可以回答此问题，尽管效率低下。

```kusto
T 
| where EventType == 'A'
| project SessionId, Start=Timestamp
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Timestamp
    ) on SessionId
| where (End - Start) between (0min .. 1min)
| project SessionId, Start, End 

```

|SessionId|开始|结束|
|---|---|---|
|0|2017-10-01 00:00:00.0000000|2017-10-01 00:01:00.0000000|

为了优化此查询，我们可以进行重写，将时间范围表示为联接键，如下所述。

**重写查询以考虑时间范围**

重写查询，将 `datetime` 值“离散化”为大小为时间范围大小一半的 Bucket。 使用 Kusto 的 *`equi-join`* 来比较这些 Bucket ID。

```kusto
let lookupWindow = 1min;
let lookupBin = lookupWindow / 2.0; // lookup bin = equal to 1/2 of the lookup window
T 
| where EventType == 'A'
| project SessionId, Start=Timestamp,
          // TimeKey on the left side of the join is mapped to a discrete time axis for the join purpose
          TimeKey = bin(Timestamp, lookupBin)
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Timestamp,
              // TimeKey on the right side of the join - emulates event 'B' appearing several times
              // as if it was 'replicated'
              TimeKey = range(bin(Timestamp-lookupWindow, lookupBin),
                              bin(Timestamp, lookupBin),
                              lookupBin)
    // 'mv-expand' translates the TimeKey array range into a column
    | mv-expand TimeKey to typeof(datetime)
    ) on SessionId, TimeKey 
| where (End - Start) between (0min .. lookupWindow)
| project SessionId, Start, End 
```

|SessionId|开始|结束|
|---|---|---|
|0|2017-10-01 00:00:00.0000000|2017-10-01 00:01:00.0000000|

**可运行的查询引用（包含内联表）**

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let T = datatable(SessionId:string, EventType:string, Timestamp:datetime)
[
    '0', 'A', datetime(2017-10-01 00:00:00),
    '0', 'B', datetime(2017-10-01 00:01:00),
    '1', 'B', datetime(2017-10-01 00:02:00),
    '1', 'A', datetime(2017-10-01 00:03:00),
    '3', 'A', datetime(2017-10-01 00:04:00),
    '3', 'B', datetime(2017-10-01 00:10:00),
];
let lookupWindow = 1min;
let lookupBin = lookupWindow / 2.0;
T 
| where EventType == 'A'
| project SessionId, Start=Timestamp, TimeKey = bin(Timestamp, lookupBin)
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Timestamp,
              TimeKey = range(bin(Timestamp-lookupWindow, lookupBin),
                              bin(Timestamp, lookupBin),
                              lookupBin)
    | mv-expand TimeKey to typeof(datetime)
    ) on SessionId, TimeKey 
| where (End - Start) between (0min .. lookupWindow)
| project SessionId, Start, End 
```

|SessionId|开始|结束|
|---|---|---|
|0|2017-10-01 00:00:00.0000000|2017-10-01 00:01:00.0000000|


**50M 数据查询**

下一个查询模拟包含 50M 记录和大约 10M ID 的一个数据集，运行该查询的方法如上所述。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let T = range x from 1 to 50000000 step 1
| extend SessionId = rand(10000000), EventType = rand(3), Time=datetime(2017-01-01)+(x * 10ms)
| extend EventType = case(EventType <= 1, "A",
                          EventType <= 2, "B",
                          "C");
let lookupWindow = 1min;
let lookupBin = lookupWindow / 2.0;
T 
| where EventType == 'A'
| project SessionId, Start=Time, TimeKey = bin(Time, lookupBin)
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Time, 
              TimeKey = range(bin(Time-lookupWindow, lookupBin), 
                              bin(Time, lookupBin),
                              lookupBin)
    | mv-expand TimeKey to typeof(datetime)
    ) on SessionId, TimeKey 
| where (End - Start) between (0min .. lookupWindow)
| project SessionId, Start, End 
| count 
```

|计数|
|---|
|1276|
