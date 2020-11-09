---
title: union 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 union 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 87abb6ca9cbfade05e56019c2edc0f6d82da4471
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105774"
---
# <a name="union-operator"></a>union 运算符

获取两个或多个表，并返回表中的所有行。 

```kusto
Table1 | union Table2, Table3
```

## <a name="syntax"></a>语法

*T* `| union` [ *UnionParameters* ] [`kind=` `inner`|`outer`] [`withsource=`*ColumnName* ] [`isfuzzy=` `true`|`false`] *Table* [`,` *Table* ]...  

不带管道输入的替代形式：

`union` [ *UnionParameters* ] [`kind=` `inner`|`outer`] [`withsource=`*ColumnName* ] [`isfuzzy=` `true`|`false`] *Table* [`,` *Table* ]...  

## <a name="arguments"></a>参数

::: zone pivot="azuredataexplorer"

* `Table`:
    *  表的名称，例如 `Events`；或
    *  必须用圆括号括起来的查询表达式，（例如 `(Events | where id==42)` 或 `(cluster("https://help.kusto.chinacloudapi.cn:443").database("Samples").table("*"))`）；或
    *  使用通配符指定的一组表。 例如，`E*` 将构成数据库中所有名称以 `E` 开头的表的并集。
* `kind`: 
    * `inner` - 此结果包含所有输入表所共有列的子集。
    * `outer` -（默认值）。 结果包含所有输入中出现的所有列。 未由输入行定义的单元格设置为 `null`。
* `withsource`=*ColumnName* ：如果指定了此项，则输出将包括一个名为 ColumnName 的列，其值指示哪个源表提供了每一行。
如果查询（在通配符匹配后）有效地引用多个数据库（默认数据库总计在内）中的表，则此列的值将具有使用数据库进行限定的表名。
同样，如果引用了多个群集，则值中将存在群集和数据库限定条件。 
* `isfuzzy=` `true` | `false`：如果 `isfuzzy` 设置为 `true` - 则允许对 union 支线进行模糊解析。 `Fuzzy` 适用于 `union` 源集。 这意味着，在分析查询和准备执行的过程中，union 源的集合被缩减为当时存在并且可供访问的一组表引用。 如果至少找到了一个这样的表，则任何解析失败都将在查询状态结果中产生警告（每个缺少的引用都会产生一个），但不会阻止查询执行；如果没有任何解析成功，则查询会返回错误。
默认为 `isfuzzy=` `false`。
* *UnionParameters* ：零个或零个以上（以空格分隔）以 Name `=` Value 形式表示的参数，用于控制行匹配操作和执行计划的行为 。 支持以下参数： 

  |名称           |值                                        |说明                                  |
  |---------------|----------------------------------------------|---------------------------------------------|
  |`hint.concurrency`|*数字*|提示系统应并行执行 `union` 运算符的多少个并发子查询。 *默认* ：群集的单个节点上的 CPU 核心数（2 到 16 个）。|
  |`hint.spread`|*数字*|提示系统并发的 `union` 子查询执行应使用多少个节点。 *默认* ：1.|

::: zone-end

::: zone pivot="azuremonitor"

* `Table`:
    *  表的名称，例如 `Events`
    *  必须用圆括号括起来的查询表达式，例如 `(Events | where id==42)`
    *  使用通配符指定的一组表。 例如，`E*` 将构成数据库中所有名称以 `E` 开头的表的并集。
* `kind`: 
    * `inner` - 此结果包含所有输入表所共有列的子集。
    * `outer` -（默认值）。 结果包含所有输入中出现的所有列。 未由输入行定义的单元格设置为 `null`。
* `withsource`=*ColumnName* ：如果指定了此项，则输出将包括一个名为 ColumnName 的列，其值指示哪个源表提供了每一行。
如果查询（在通配符匹配后）有效地引用多个数据库（默认数据库总计在内）中的表，则此列的值将具有使用数据库进行限定的表名。
同样，如果引用了多个群集，则值中将存在群集和数据库限定条件。 
* `isfuzzy=` `true` | `false`：如果 `isfuzzy` 设置为 `true` - 则允许对 union 支线进行模糊解析。 `Fuzzy` 适用于 `union` 源集。 这意味着，在分析查询和准备执行的过程中，union 源的集合被缩减为当时存在并且可供访问的一组表引用。 如果至少找到了一个这样的表，则任何解析失败都将在查询状态结果中产生警告（每个缺少的引用都会产生一个），但不会阻止查询执行；如果没有任何解析成功，则查询会返回错误。
默认为 `isfuzzy=false`。

::: zone-end

## <a name="returns"></a>返回

一个表，其中的行数与所有输入表中的行数相同。

**备注**

::: zone pivot="azuredataexplorer"

1. `union` 作用域可以包括 [let 语句](./letstatement.md)（如果这些语句通过 [view 关键字](./letstatement.md)进行了特性化）
2. `union` 作用域将不包括[函数](../management/functions.md)。 若要在 union 作用域中包括某个函数，请定义带 [view 关键字](./letstatement.md)的 [let 语句](./letstatement.md)
3. 如果 `union` 输入是[表](../management/tables.md)（与[表格表达式](./tabularexpressionstatements.md)相对），并且 `union` 后跟一个 [where 运算符](./whereoperator.md)，则为获得更好的性能，请考虑使用 [find](./findoperator.md) 来替代两者。 请注意 `find` 运算符生成的不同[输出架构](./findoperator.md#output-schema)。 
4. `isfuzzy=true` 仅适用于 `union` 源解析阶段。 确定源表的集合后，不会隐藏可能的其他查询失败。
5. 使用 `outer union` 时，结果将包含出现在任何输入中的所有列，每个名称和类型实例各占一列。 这意味着，如果某个列出现在多个表中且具有多个类型，则在 `union` 的结果中，对于每个类型，它都有一个对应的列。 此列名称将以“_”作为后缀，后跟源列[类型](./scalar-data-types/index.md)。

::: zone-end

::: zone pivot="azuremonitor"

1. `union` 作用域可以包括 [let 语句](./letstatement.md)（如果这些语句通过 [view 关键字](./letstatement.md)进行了特性化）
2. `union` 作用域将不包括函数。 若要在 union 作用域中包括函数，请定义带 [view 关键字](./letstatement.md)的 [let 语句](./letstatement.md)
3. 如果 `union` 输入是表（与[表格表达式](./tabularexpressionstatements.md)相对），并且 `union` 后跟一个 [where 运算符](./whereoperator.md)，则为获得更好的性能，请考虑使用 [find](./findoperator.md) 来替代两者。 请注意 `find` 运算符生成的不同[输出架构](./findoperator.md#output-schema)。 
4. `isfuzzy=` `true` 仅适用于 `union` 源解析的阶段。 确定源表的集合后，不会隐藏可能的其他查询失败。
5. 使用 `outer union` 时，结果将包含出现在任何输入中的所有列，每个名称和类型实例各占一列。 这意味着，如果某个列出现在多个表中且具有多个类型，则在 `union` 的结果中，对于每个类型，它都有一个对应的列。 此列名称将以“_”作为后缀，后跟源列[类型](./scalar-data-types/index.md)。

::: zone-end


## <a name="example-tables-with-string-in-name-or-column"></a>示例：名称或列中包含字符串的表

```kusto
union K* | where * has "Kusto"
```

数据库中名称以 `K` 开头的所有表中有任意列包括单词 `Kusto` 的行。

## <a name="example-distinct-count"></a>示例：非重复计数

```kusto
union withsource=SourceTable kind=outer Query, Command
| where Timestamp > ago(1d)
| summarize dcount(UserId)
```

过去一天已生成 `Query` 事件或 `Command` 事件的不同用户数。 在结果中，“SourceTable”列会指示“Query”或“Command”。

```kusto
Query
| where Timestamp > ago(1d)
| union withsource=SourceTable kind=outer 
   (Command | where Timestamp > ago(1d))
| summarize dcount(UserId)
```

此效率更高的版本可生成相同结果。 它可在创建联合之前筛选每个表。

**示例：使用 `isfuzzy=true`**
 
```kusto     
// Using union isfuzzy=true to access non-existing view:                                     
let View_1 = view () { print x=1 };
let View_2 = view () { print x=1 };
let OtherView_1 = view () { print x=1 };
union isfuzzy=true
(View_1 | where x > 0), 
(View_2 | where x > 0),
(View_3 | where x > 0)
| count 
```

|计数|
|---|
|2|

观察查询状态 - 返回以下警告：`Failed to resolve entity 'View_3'`

```kusto
// Using union isfuzzy=true and wildcard access:
let View_1 = view () { print x=1 };
let View_2 = view () { print x=1 };
let OtherView_1 = view () { print x=1 };
union isfuzzy=true View*, SomeView*, OtherView*
| count 
```

|计数|
|---|
|3|

观察查询状态 - 返回以下警告：`Failed to resolve entity 'SomeView*'`

**示例：源列类型不匹配**
 
```kusto     
let View_1 = view () { print x=1 };
let View_2 = view () { print x=toint(2) };
union withsource=TableName View_1, View_2
```

|TableName|x_long|x_int|
|---------|------|-----|
|View_1   |1     |     |
|View_2   |      |2    |

```kusto     
let View_1 = view () { print x=1 };
let View_2 = view () { print x=toint(2) };
let View_3 = view () { print x_long=3 };
union withsource=TableName View_1, View_2, View_3 
```

|TableName|x_long1|x_int |x_long|
|---------|-------|------|------|
|View_1   |1      |      |      |
|View_2   |       |2     |      |
|View_3   |       |      |3     |

来自 `View_1` 的列 `x` 收到了后缀 `_long`，并且因为结果架构中已存在名为 `x_long` 的列，所以对列名称执行了删除重复数据的操作，生成了新列 - `x_long1`
