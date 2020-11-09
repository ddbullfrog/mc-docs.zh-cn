---
title: partition 运算符 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 partition 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 0b263225a4a57ded3a332d176eb776a655e86711
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105032"
---
# <a name="partition-operator"></a>partition 运算符

partition 运算符根据指定列的值将其输入表分区为多个子表，对每个子表执行子查询，并生成一个输出表，该表是所有子查询的结果的并集。 

```kusto
T | partition by Col1 ( top 10 by MaxValue )

T | partition by Col1 { U | where Col2=toscalar(Col1) }
```

## <a name="syntax"></a>语法

*T* `|` `partition` [ *PartitionParameters* ] `by` *Column* `(` *ContextualSubquery* `)`

*T* `|` `partition` [ *PartitionParameters* ] `by` *Column* `{` *Subquery* `}`

## <a name="arguments"></a>参数

* *T* ：要由运算符处理其数据的表格源。

* *Column* ：T 中某个列的名称，其值决定了如何对输入表进行分区。 请参阅下文中的 **说明** 。

* ContextualSubquery：一个表格表达式，其源是 `partition` 运算符的源，其作用域为单个键值。

* Subquery：没有源的表格表达式。 键值可通过 `toscalar()` 调用获得。

* PartitionParameters：零个或零个以上的（使用空格进行分隔的）参数，它们控制运算符的行为，格式为： *Name* `=` *Value* 。 支持以下参数：

  |名称               |值         |说明|
  |-------------------|---------------|-----------|
  |`hint.materialized`|`true`,`false` |如果设置为 `true`，将对 `partition` 运算符的源进行具体化（默认值：`false`）|
  |`hint.concurrency`|*数字*|提示系统应并行执行 `partition` 运算符的多少个并发子查询。 *默认* ：群集的单个节点上的 CPU 核心数（2 到 16 个）。|
  |`hint.spread`|*数字*|提示系统并发的 `partition` 子查询执行应使用多少个节点。 *默认* ：1.|

## <a name="returns"></a>返回

此运算符返回对输入数据的每个分区应用子查询得到的结果的并集。

**备注**

* partition 运算符当前受分区数的限制。
  最多可以创建 64 个不同的分区。
  如果分区列（Column）的非重复值超过 64 个，则该运算符会产生错误。

* 子查询隐式引用输入分区（子查询中没有分区的“名称”）。 若要在子查询中多次引用输入分区，请使用 [as 运算符](asoperator.md)，如下文中的 **示例：partition 参考** 所示。

**示例：top-nested 案例**

在某些情况下，使用 `partition` 运算符编写查询比使用 [`top-nested` 运算符](topnestedoperator.md)性能更高且更容易。下一个示例将针对以 `W` 开头的以下每个 State 运行子查询来计算 `summarize` 和 `top`：（WYOMING、WASHINGTON、WEST VIRGINIA、WISCONSIN）

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| where State startswith 'W'
| partition by State 
(
    summarize Events=count(), Injuries=sum(InjuriesDirect) by EventType, State
    | top 3 by Events 
) 

```
|EventType|状态|事件|Injuries|
|---|---|---|---|
|冰雹|怀俄明州|108|0|
|疾风|怀俄明州|81|5|
|冬季风暴|怀俄明州|72|0|
|大雪|华盛顿州|82|0|
|疾风|华盛顿州|58|13|
|野火|华盛顿州|29|0|
|雷雨大风|西弗吉尼亚州|180|1|
|冰雹|西弗吉尼亚州|103|0|
|冬季天气|西弗吉尼亚州|88|0|
|雷雨大风|威斯康星州|416|1|
|冬季风暴|威斯康星州|310|0|
|冰雹|威斯康星州|303|1|

**示例：查询非重叠数据分区**

有时候，以映射/化简样式对非重叠数据分区运行复杂子查询很有用（在性能方面）。 下面的示例展示了如何在 10 个分区上创建聚合的手动分布。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| extend p = hash(EventId, 10)
| partition by p
(
    summarize Count=count() by Source 
)
| summarize Count=sum(Count) by Source
| top 5 by Count
```

|Source|计数|
|---|---|
|专业观测员|12770|
|执法机构|8570|
|公用|6157|
|灾害管理|4900|
|COOP Observer|3039|

**示例：查询时分区**

下面的示例展示了如何将查询分区为 N = 10 个分区，其中的每个分区都计算其自己的 Count，然后全部汇总到 TotalCount。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let N = 10;                 // Number of query-partitions
range p from 0 to N-1 step 1  // 
| partition by p            // Run the sub-query partitioned 
{
    StormEvents 
    | where hash(EventId, N) == toscalar(p) // Use toscalar() to fetch partition key value
    | summarize Count = count()
}
| summarize TotalCount=sum(Count) 
```

|TotalCount|
|---|
|59066|


**示例：分区引用**

下面的示例展示了如何使用 [as 运算符](asoperator.md)为每个数据分区指定“名称”，然后在子查询中重复使用该名称：

```kusto
T
| partition by Dim
(
    as Partition
    | extend MetricPct = Metric * 100.0 / toscalar(Partition | summarize sum(Metric))
)
```

**示例：由函数调用隐藏的复杂子查询**

同样的技术可以应用于复杂得多的子查询。 若要简化语法，可以将子查询包装在函数调用中：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let partition_function = (T:(Source:string)) 
{
    T
    | summarize Count=count() by Source
};
StormEvents
| extend p = hash(EventId, 10)
| partition by p
(
    invoke partition_function()
)
| summarize Count=sum(Count) by Source
| top 5 by Count
```

|Source|计数|
|---|---|
|专业观测员|12770|
|执法机构|8570|
|公用|6157|
|灾害管理|4900|
|COOP Observer|3039|