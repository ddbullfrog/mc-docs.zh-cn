---
title: join 运算符 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的联接运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/30/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: f0b935fb3022b6ef784d1793c9e3d8e746df85af
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104611"
---
# <a name="join-operator"></a>join 运算符

通过匹配每个表中指定列的值，合并两个表的行以组成新表。

```kusto
Table1 | join (Table2) on CommonColumn, $left.Col1 == $right.Col2
```

## <a name="syntax"></a>语法

*LeftTable* `|` `join` [ *JoinParameters* ] `(` *RightTable* `)` `on` *Attributes*

## <a name="arguments"></a>参数

* LeftTable：要合并其行的 **左侧** 表或表格表达式（有时称为 **外部** 表）。 表示为 `$left`。

* RightTable：要合并其行的 **右侧** 表或表格表达式（有时称为 **内部** 表）。 表示为 `$right`。

* Attributes：一个或多个逗号分隔的规则，这些规则描述 LeftTable 中的行如何与 RightTable 中的行进行匹配。 将使用 `and` 逻辑运算符评估多个规则。

  **规则** 可以是下列项之一：

  |规则类型        |语法          |Predicate    |
  |-----------------|--------------|-------------------------|
  |基于名称的等式 |*ColumnName*    |`where` *LeftTable*. *ColumnName* `==` *RightTable*. *ColumnName*|
  |基于值的等式|`$left.`*LeftColumn* `==` `$right.`*RightColumn*|`where` `$left.`*LeftColumn* `==` `$right.`*RightColumn*       |

    > [!NOTE]
    > 如果使用“基于值的等式”，则列名称必须通过由 `$left` 和 `$right` 表示法表示的相应的所有者表进行限定。

* *JoinParameters* ：零个或零个以上（以空格分隔）以 Name `=` Value 形式表示的参数，用于控制行匹配操作和执行计划的行为 。 支持以下参数：

    ::: zone pivot="azuredataexplorer"

    |参数名称           |值                                        |说明                                  |
    |---------------|----------------------------------------------|---------------------------------------------|
    |`kind`         |联接风格|请参阅[联接风格](#join-flavors)|                                             |
    |`hint.remote`  |`auto`, `left`, `local`, `right`              |请参阅[跨群集联接](joincrosscluster.md)|
    |`hint.strategy`|执行提示                               |请参阅[联接提示](#join-hints)                |

    ::: zone-end

    ::: zone pivot="azuremonitor"

    |名称           |值                                        |说明                                  |
    |---------------|----------------------------------------------|---------------------------------------------|
    |`kind`         |联接风格|请参阅[联接风格](#join-flavors)|                                             |
    |`hint.remote`  |`auto`, `left`, `local`, `right`              |                                             |
    |`hint.strategy`|执行提示                               |请参阅[联接提示](#join-hints)                |

    ::: zone-end

> [!WARNING]
> 如果未指定 `kind`，则默认的联接风格为 `innerunique`。 这不同于其他的一些分析产品，这些产品以 `inner` 作为默认风格。  请参阅[联接风格](#join-flavors)来了解不同之处，确保查询产生预期结果。

## <a name="returns"></a>返回

**输出架构取决于联接风格：**

| 联接风格 | 输出架构 |
|---|---|
|`kind=leftanti`, `kind=leftsemi`| 结果表中只包含来自左侧的列。|
| `kind=rightanti`, `kind=rightsemi` | 结果表中只包含来自右侧的列。|
|  `kind=innerunique`, `kind=inner`, `kind=leftouter`, `kind=rightouter`, `kind=fullouter` |  一列，用于每个表中的每一列，包括匹配键。 如果存在名称冲突，会自动重命名右侧列。 |
   
**输出记录取决于联接风格：**

   > [!NOTE]
   > 如果这些字段有多个行具有相同的值，则会获得所有合并的行。
   > 匹配项是从表中选出的一行，该表中的所有 `on` 字段值与其他表中的值相同。

| 联接风格 | 输出记录 |
|---|---|
|`kind=leftanti`, `kind=leftantisemi`| 返回左侧中在右侧没有匹配项的所有记录|
| `kind=rightanti`, `kind=rightantisemi`| 返回右侧中在左侧没有匹配项的所有记录。|
| `kind` 未指定，`kind=innerunique`| 左侧中仅有一行与 `on` 键的每个值匹配。 输出包含一行，用于此行与右侧行的每一个匹配项。|
| `kind=leftsemi`| 返回左侧中在右侧具有匹配项的所有记录。 |
| `kind=rightsemi`| 返回右侧中在左侧具有匹配项的所有记录。 |
|`kind=inner`| 输出中包含一行，该行对应于左右匹配行的每种组合。 |
| `kind=leftouter`（或 `kind=rightouter` 或 `kind=fullouter`）| 包含的一行对应于左侧和右侧的每一行，即使没有匹配项。 不匹配的输出单元格包含 null。 |

> [!TIP]
> 为获得最佳性能，如果某个表始终小于另一个表，则将其用作 join 的左侧（管接）。

## <a name="example"></a>示例

从 `login` 中获取扩展活动，某些条目将其标记为活动的开始和结束。

```kusto
let Events = MyLogTable | where type=="Event" ;
Events
| where Name == "Start"
| project Name, City, ActivityId, StartTime=timestamp
| join (Events
    | where Name == "Stop"
        | project StopTime=timestamp, ActivityId)
    on ActivityId
| project City, ActivityId, StartTime, StopTime, Duration = StopTime - StartTime
```

```kusto
let Events = MyLogTable | where type=="Event" ;
Events
| where Name == "Start"
| project Name, City, ActivityIdLeft = ActivityId, StartTime=timestamp
| join (Events
        | where Name == "Stop"
        | project StopTime=timestamp, ActivityIdRight = ActivityId)
    on $left.ActivityIdLeft == $right.ActivityIdRight
| project City, ActivityId, StartTime, StopTime, Duration = StopTime - StartTime
```

## <a name="join-flavors"></a>联接风格

join 运算符的确切风格是通过 kind 关键字指定的。 支持 join 运算符的以下风格：

|联接类型/风格|描述|
|--|--|
|[`innerunique`](#default-join-flavor)（或默认为空）|执行左侧重复数据删除的内联|
|[`inner`](#inner-join-flavor)|标准内联|
|[`leftouter`](#left-outer-join-flavor)|左外部联接|
|[`rightouter`](#right-outer-join-flavor)|右外部联接|
|[`fullouter`](#full-outer-join-flavor)|完全外联|
|[`leftanti`](#left-anti-join-flavor)、[`anti`](#left-anti-join-flavor) 或 [`leftantisemi`](#left-anti-join-flavor)|左反联|
|[`rightanti`](#right-anti-join-flavor) 或 [`rightantisemi`](#right-anti-join-flavor)|右反联|
|[`leftsemi`](#left-semi-join-flavor)|左半联|
|[`rightsemi`](#right-semi-join-flavor)|右半联|

### <a name="default-join-flavor"></a>默认联接风格

默认联接风格是在左侧删除了重复数据的内联。 在典型的日志/跟踪分析方案中，默认联接实现非常有用。在这种方案中，你想要关联两个事件，每个事件都在同一个相关 ID 下匹配某个筛选条件。 你需要获取所有出现的现象，忽略多次出现的构成跟踪记录。

``` 
X | join Y on Key
 
X | join kind=innerunique Y on Key
```

下面的两个示例表用来说明联接操作。

**表 X**

|键 |Value1
|---|---
|a |1
|b |2
|b |3
|c |4

**表 Y**

|键 |Value2
|---|---
|b |10
|c |20
|c |30
|d |40

默认联接在左侧对联接键执行重复数据删除后（删除重复数据时会保留第一个记录）执行内联。

对于以下语句：`X | join Y on Key`

联接的有效左侧（删除重复数据后的表 X）将是：

|键 |Value1
|---|---
|a |1
|b |2
|c |4

联接结果将是：

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join Y on Key
```

|键|Value1|Key1|Value2|
|---|---|---|---|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|

> [!NOTE]
> 键“a”和“d”没有出现在输出中，因为在左侧和右侧都没有匹配的键。

### <a name="inner-join-flavor"></a>内联风格

内联函数类似于 SQL 中的标准内联。 只要左侧的记录具有与右侧记录相同的联接键，就会生成输出记录。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=inner Y on Key
```

|键|Value1|Key1|Value2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|

> [!NOTE]
> * 右侧的 (b,10) 联接了两次：与左侧的 (b,2) 和 (b,3) 都进行了联接。
> * 左侧的 (c,4) 联接了两次：与右侧的 (c,20) 和 (c,30) 都进行了联接。

### <a name="innerunique-join-flavor"></a>Innerunique 联接风格
 
使用 **innerunique 联接风格** 从左侧删除重复键。 结果将是进行了重复数据删除的左键和右键的每种组合在输出中存在一行。

> [!NOTE]
> **innerunique 风格** 可能产生两个可能的输出，两者都是正确的。
在第一个输出中，join 运算符随机选择了出现在 t1 中的第一个键，其值为“val1.1”，并将其与 t2 键匹配。
在第二个输出中，join 运算符随机选择了出现在 t1 中的第二个键，其值为“val1.2”，并将其与 t2 键匹配。

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3",
1, "val1.4"  
];
t1
| join kind = innerunique
    t2
on key
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.1|1|val1.3|
|1|val1.1|1|val1.4|

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3", 
1, "val1.4"  
];
t1
| join kind = innerunique
    t2
on key
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.2|1|val1.3|
|1|val1.2|1|val1.4|

* Kusto 经过优化，它会尽可能将 `join` 之后的筛选器推向相应的联接端，不管是左侧还是右侧。

* 有时，使用的风格是 **innerunique** ，并且筛选器将传播到联接的左侧。 风格会自动传播，应用于该筛选器的键会始终出现在输出中。
    
* 使用上面的示例，添加筛选器 `where value == "val1.2" `。 它将始终提供第二个结果，永远不会为数据集提供第一个结果：

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3", 
1, "val1.4"  
];
t1
| join kind = innerunique
    t2
on key
| where value == "val1.2"
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.2|1|val1.3|
|1|val1.2|1|val1.4|

### <a name="left-outer-join-flavor"></a>左外部联接风格

表 X 和 Y 的左外部联接的结果始终包含左表 (X) 的所有记录，即使联接条件在右表 (Y) 中未找到任何匹配记录。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=leftouter Y on Key
```

|键|Value1|Key1|Value2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|
|a|1|||

### <a name="right-outer-join-flavor"></a>右外部联接风格

右外部联接风格与左外部联接类似，但对表的处理是相反的。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=rightouter Y on Key
```

|键|Value1|Key1|Value2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|
|||d|40|

### <a name="full-outer-join-flavor"></a>完全外部联接风格

完全外部联接合并了应用左外部联接和右外部联接的效果。 如果联接的表中的记录不匹配，则对于表中缺少匹配行的每个列，结果集都会有 `null` 值。 对于那些匹配的记录，结果集中会生成单个行（其中包含两个表中填充的字段）。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=fullouter Y on Key
```

|键|Value1|Key1|Value2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|
|||d|40|
|a|1|||

### <a name="left-anti-join-flavor"></a>左反联接风格

左反联接返回左侧中在右侧没有任何匹配记录的所有记录。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=leftanti Y on Key
```

|键|Value1|
|---|---|
|a|1|

> [!NOTE]
> 反联模拟“NOT IN”查询。

### <a name="right-anti-join-flavor"></a>右反联接风格

右反联接返回右侧中在左侧没有任何匹配记录的所有记录。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=rightanti Y on Key
```

|键|Value2|
|---|---|
|d|40|

> [!NOTE]
> 反联模拟“NOT IN”查询。

### <a name="left-semi-join-flavor"></a>左半联接风格

左半联接返回左侧中在右侧具有匹配记录的所有记录。 仅会返回左侧的列。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=leftsemi Y on Key
```

|键|Value1|
|---|---|
|b|3|
|b|2|
|c|4|

### <a name="right-semi-join-flavor"></a>右半联接风格

右半联接返回右侧中在左侧具有匹配记录的所有记录。 仅会返回右侧的列。

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=rightsemi Y on Key
```

|键|Value2|
|---|---|
|b|10|
|c|20|
|c|30|

### <a name="cross-join"></a>交叉联接

Kusto 本身不提供交叉联接风格。 无法用 `kind=cross` 来标记运算符。
若要进行模拟，请使用虚拟键。

`X | extend dummy=1 | join kind=inner (Y | extend dummy=1) on dummy`

## <a name="join-hints"></a>联接提示

`join` 运算符支持许多用于控制查询运行方式的提示。
这些提示不会更改 `join` 的语义，但可能会影响其性能。

以下文章解释了联接提示：

* `hint.shufflekey=<key>` 和 `hint.strategy=shuffle` - [随机执行查询](shufflequery.md)
* `hint.strategy=broadcast` - [广播联接](broadcastjoin.md)
* `hint.remote=<strategy>` - [跨群集联接](joincrosscluster.md)
