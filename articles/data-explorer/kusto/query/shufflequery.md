---
title: 随机执行查询 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的随机执行查询。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 0b9a4ec03a2e9564f833063c5dad49dc16ffcc94
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104262"
---
# <a name="shuffle-query"></a>随机执行查询

随机执行查询是一种保留语义的转换，适用于支持随机执行策略的一组运算符。 此查询可以显著提高性能，具体取决于实际数据。

Kusto 中支持随机执行的运算符是 [join](joinoperator.md)、[summarize](summarizeoperator.md) 和 [make-series](make-seriesoperator.md)。

可以使用查询参数 `hint.strategy = shuffle` 或 `hint.shufflekey = <key>` 设置随机执行查询策略。

## <a name="syntax"></a>语法

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.strategy = shuffle (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

```kusto
T
| summarize hint.strategy = shuffle count(), avg(price) by supplier
```

```kusto
T
| make-series hint.shufflekey = Fruit PriceAvg=avg(Price) default=0  on Purchase from datetime(2016-09-10) to datetime(2016-09-13) step 1d by Supplier, Fruit
```

此策略会在所有群集节点上共享负载，其中的每个节点将处理一个数据分区。
如果键（`join` 键、`summarize` 键或 `make-series` 键）的基数较高且常规查询策略达到查询限制，则使用随机执行查询策略会很有用。

**hint.strategy=shuffle 与 hint.shufflekey = key 之间的差异**

`hint.strategy=shuffle` 表示随机执行的运算符的执行顺序会被所有键打乱。
例如，在以下查询中：

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.strategy = shuffle (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId, ProcessId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

随机操作数据的哈希函数会同时使用 ActivityId 键和 ProcessId 键。

上面的查询等效于：

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.shufflekey = ActivityId hint.shufflekey = ProcessId (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId, ProcessId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

如果复合键足够唯一，但每个键都不够唯一，请使用此 `hint` 按随机执行的运算符的所有键随机操作数据。
如果随机执行的运算符具有其他能够随机执行的运算符（例如 `summarize` 或 `join`），则查询会变得更加复杂，那么就不会应用 hint.strategy=shuffle。

例如：

```kusto
T
| where Event=="Start"
| project ActivityId, Started=Timestamp, numeric_column
| summarize count(), numeric_column = any(numeric_column) by ActivityId
| join
    hint.strategy = shuffle (T
    | where Event=="End"
    | project ActivityId, Ended=Timestamp, numeric_column
)
on ActivityId, numeric_column
| extend Duration=Ended - Started
| summarize avg(Duration)
```

如果你应用 `hint.strategy=shuffle`（而不是在查询规划期间忽略此策略），并根据复合键 [`ActivityId`, `numeric_column`] 随机操作数据，则结果将不正确。
`summarize` 运算符位于 `join` 运算符的左侧。 此运算符将按 `join` 键的子集（在我们的示例中为 `ActivityId`）进行分组。 因此，`summarize` 将按 `ActivityId` 键进行分组，而数据将按复合键 [`ActivityId`, `numeric_column`] 进行分区。
按复合键 [`ActivityId`, `numeric_column`] 随机执行不一定意味着针对 `ActivityId` 键的随机执行有效，并且结果可能不正确。

此示例假设用于复合键的哈希函数是 `binary_xor(hash(key1, 100) , hash(key2, 100))`：

```kusto

datatable(ActivityId:string, NumericColumn:long)
[
"activity1", 2,
"activity1" ,1,
]
| extend hash_by_key = binary_xor(hash(ActivityId, 100) , hash(NumericColumn, 100))
```

|ActivityId|NumericColumn|hash_by_key|
|---|---|---|
|activity1|2|56|
|activity1|1|65|

这两条记录的复合键映射到不同的分区（56 和 65），但这两条记录具有相同的值 `ActivityId`。 `join` 左侧的 `summarize` 运算符预期 `ActivityId` 列的类似值将位于同一分区中。 此查询会产生不正确的结果。

可以通过使用 `hint.shufflekey` 将联接的随机执行键指定为 `hint.shufflekey = ActivityId` 来解决此问题。 此键是所有能够随机执行的运算符通用的。
在本例中，随机执行是安全的，因为 `join` 和 `summarize` 按同一键随机执行。 因此，所有类似的值都将位于同一个分区中，结果是正确的：

```kusto
T
| where Event=="Start"
| project ActivityId, Started=Timestamp, numeric_column
| summarize count(), numeric_column = any(numeric_column) by ActivityId
| join
    hint.shufflekey = ActivityId (T
    | where Event=="End"
    | project ActivityId, Ended=Timestamp, numeric_column
)
on ActivityId, numeric_column
| extend Duration=Ended - Started
| summarize avg(Duration)
```

|ActivityId|NumericColumn|hash_by_key|
|---|---|---|
|activity1|2|56|
|activity1|1|65|

在随机执行查询中，默认分区数是群集节点数。 可以使用 `hint.num_partitions = total_partitions` 语法来重写此数目，该语法将控制分区数。

如果群集具有少量的群集节点且其中的默认分区数也较小，而查询仍然失败或需要较长的执行时间，则此提示非常有用。

> [!Note]
> 具有许多分区可能会消耗更多的群集资源并降低性能。 应改为从 hint.strategy = shuffle 着手，谨慎选择分区数，然后逐渐开始增加分区。

## <a name="examples"></a>示例

下面的示例展示了随机执行 `summarize` 如何显著提高性能。

源表有 150M 记录，分组依据键的基数是 10M，它分布在 10 个群集节点上。

运行常规 `summarize` 策略时，查询将在 1 分 8 秒之后结束，内存使用量峰值大约为 3 GB：

```kusto
orders
| summarize arg_max(o_orderdate, o_totalprice) by o_custkey 
| where o_totalprice < 1000
| count
```

|计数|
|---|
|1086|

使用随机执行 `summarize` 策略时，查询将在大约 7 秒之后结束，内存使用量峰值为 0.43 GB：

```kusto
orders
| summarize hint.strategy = shuffle arg_max(o_orderdate, o_totalprice) by o_custkey 
| where o_totalprice < 1000
| count
```

|计数|
|---|
|1086|

下面的示例展示了对具有两个群集节点的群集的改进，表中包含 60M 记录，分组依据键的基数为 2M。

在不使用 `hint.num_partitions` 的情况下运行查询将只使用两个分区（作为群集节点数），以下查询将需要大约 1 分 10 秒的时间：

```kusto
lineitem    
| summarize hint.strategy = shuffle dcount(l_comment), dcount(l_shipdate) by l_partkey 
| consume
```

如果将分区数设置为 10，则查询将在 23 秒后结束： 

```kusto
lineitem    
| summarize hint.strategy = shuffle hint.num_partitions = 10 dcount(l_comment), dcount(l_shipdate) by l_partkey 
| consume
```

下面的示例展示了随机执行 `join` 如何显著提高性能。

示例是在包含 10 个节点的群集上采样的，数据分散在所有这些节点上。

左表有 15M 记录，其中 `join` 键的基数大约为 14M。 `join` 的右侧有 150M 记录，而 `join` 键的基数为 10M。
运行 `join` 的常规策略时，查询将在大约 28 秒之后结束，内存使用量峰值大约为 1.43 GB：

```kusto
customer
| join
    orders
on $left.c_custkey == $right.o_custkey
| summarize sum(c_acctbal) by c_nationkey
```

使用随机执行 `join` 策略时，查询将在大约 4 秒之后结束，内存使用量峰值为 0.3 GB：

```kusto
customer
| join
    hint.strategy = shuffle orders
on $left.c_custkey == $right.o_custkey
| summarize sum(c_acctbal) by c_nationkey
```

尝试对更大的数据集执行相同的查询，其中，`join` 的左侧为 150M，键的基数为 148M。 `join` 的右侧为 1.5B，而键的基数大约为 100M。

使用默认的 `join` 策略的查询会达到 Kusto 限制并且在 4 分钟后超时。
使用随机执行 `join` 策略时，查询将在大约 34 秒之后结束，内存使用量峰值为 1.23 GB。


下面的示例展示了对具有两个群集节点的群集的改进，表中包含 60M 记录，`join` 键的基数为 2M。
在不使用 `hint.num_partitions` 的情况下运行查询将只使用两个分区（作为群集节点数），以下查询将需要大约 1 分 10 秒的时间：

```kusto
lineitem
| summarize dcount(l_comment), dcount(l_shipdate) by l_partkey
| join
    hint.shufflekey = l_partkey   part
on $left.l_partkey == $right.p_partkey
| consume
```

如果将分区数设置为 10，则查询将在 23 秒后结束： 

```kusto
lineitem
| summarize dcount(l_comment), dcount(l_shipdate) by l_partkey
| join
    hint.shufflekey = l_partkey  hint.num_partitions = 10    part
on $left.l_partkey == $right.p_partkey
| consume
```
