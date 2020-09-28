---
title: mv-apply 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 mv-apply 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 9730e407a9b133685cc8bb9d039e0cf3681afe1e
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146781"
---
# <a name="mv-apply-operator"></a>mv-apply 运算符

对每个记录应用子查询，并返回所有子查询结果的并集。

例如，假设表 `T` 包含 `dynamic` 类型的列 `Metric`，其值为 `real` 数组成的数组。 下面的查询将在每个 `Metric` 值中找到两个最大值，并返回对应于这些值的记录。

```kusto
T | mv-apply Metric to typeof(real) on 
(
   top 2 by Metric desc
)
```

`mv-apply` 运算符包括以下处理步骤：

1. 使用 [`mv-expand`](./mvexpandoperator.md) 运算符将输入中的每条记录扩展为子表。
1. 为每个子表应用子查询。
1. 将零个或更多列添加到生成的子表。 这些列包含未扩展的源列的值，并在需要时重复。
1. 返回结果的并集。

`mv-apply` 运算符获取以下输入：

1. 一个或多个表达式，其计算结果为要扩展的动态数组。
   每个扩展子表中的记录数是每个动态数组的最大长度。 如果指定了多个表达式，且相应数组的长度不同，则会添加 NULL 值。

1. （可选）用于在扩展后分配表达式值的名称。
   这些名称将成为子表中的列名。
   如果未指定，则在表达式为列引用时使用列的原始名称。 否则，使用随机名称。 

   > [!NOTE]
   > 建议使用默认列名。

1. 扩展后，这些动态数组的元素的数据类型。
   它们将成为子表中列的列类型。
   如果未指定，则使用 `dynamic`。

1. （可选）要添加到子表中的列名，该列指定生成子表记录的数组中元素的从 0 开始的索引。

1. （可选）要扩展的最大数组元素数。

可以将 `mv-apply` 运算符视为 [`mv-expand`](./mvexpandoperator.md) 运算符的通用化（事实上，如果子查询只包含投影，则后者可以通过前者来实现。）

## <a name="syntax"></a>语法

T `|` `mv-apply` [ItemIndex] ColumnsToExpand [RowLimit] `on` `(` SubQuery `)`    

其中 ItemIndex 的语法如下：

`with_itemindex` `=` IndexColumnName

ColumnsToExpand 是以逗号分隔的列表，其中列出了一个或多个以下形式的元素：

[Name `=`] ArrayExpression [`to` `typeof` `(`Typename`)`]  

RowLimit 只是：

`limit` RowLimit

SubQuery 具有与任何查询语句相同的语法。

## <a name="arguments"></a>参数

* ItemIndex：如果使用，则指示 `long` 类型的列的名称，该列在数组扩展阶段追加到输入，并指示扩展值的从 0 开始的数组索引。

* *名称*：如果使用，则为用于分配每个数组扩展表达式的数组扩展值的名称。
  如果未指定，则使用列名（如果可用）。
  如果 ArrayExpression 不是简单的列名，则生成随机名称。

* ArrayExpression：`dynamic` 类型的表达式，其值将经过数组扩展。
  如果表达式为输入中的列名，则会从输入中删除输入列，输出中将显示具有相同名称（如已指定，则为 ColumnName）的新列。

* Typename：如果使用，则为 `dynamic` 数组 ArrayExpression 的各个元素采用的类型名称。 不符合此类型的元素将替换为 NULL 值。
  （如果未指定，则默认情况下使用 `dynamic`。）

* RowLimit：如果使用，则为基于每个输入记录生成的记录数限制。
  （如果未指定，则使用 2147483647。）

* SubQuery：具有隐式表格源的表格查询表达式，应用于每个数组扩展子表。

**备注**

* 与 [`mv-expand`](./mvexpandoperator.md) 运算符不同，`mv-apply` 运算符仅支持数组扩展。 不支持扩展属性包。

## <a name="examples"></a>示例

## <a name="getting-the-largest-element-from-the-array"></a>获取数组中最大的元素

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let _data =
range x from 1 to 8 step 1
| summarize l=make_list(x) by xMod2 = x % 2;
_data
| mv-apply element=l to typeof(long) on 
(
   top 1 by element
)
```

|`xMod2`|l           |element|
|-----|------------|-------|
|1    |[1, 3, 5, 7]|7      |
|0    |[2, 4, 6, 8]|8      |

## <a name="calculating-the-sum-of-the-largest-two-elements-in-an-array"></a>计算数组中最大的两个元素的总和

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let _data =
range x from 1 to 8 step 1
| summarize l=make_list(x) by xMod2 = x % 2;
_data
| mv-apply l to typeof(long) on
(
   top 2 by l
   | summarize SumOfTop2=sum(l)
)
```

|`xMod2`|l        |SumOfTop2|
|-----|---------|---------|
|1    |[1,3,5,7]|12       |
|0    |[2,4,6,8]|14       |


## <a name="using-with_itemindex-for-working-with-a-subset-of-the-array"></a>使用 `with_itemindex` 来处理数组的子集

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let _data =
range x from 1 to 10 step 1
| summarize l=make_list(x) by xMod2 = x % 2;
_data
| mv-apply with_itemindex=index element=l to typeof(long) on 
(
   // here you have 'index' column
   where index >= 3
)
| project index, element
```

|index|element|
|---|---|
|3|7|
|4|9|
|3|8|
|4|10|

## <a name="using-the-mv-apply-operator-to-sort-the-output-of-make_list-aggregate-by-some-key"></a>使用 `mv-apply` 运算符按某个键对 `make_list` 聚合的输出进行排序

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(command:string, command_time:datetime, user_id:string)
[
    'chmod',        datetime(2019-07-15),   "user1",
    'ls',           datetime(2019-07-02),   "user1",
    'dir',          datetime(2019-07-22),   "user1",
    'mkdir',        datetime(2019-07-14),   "user1",
    'rm',           datetime(2019-07-27),   "user1",
    'pwd',          datetime(2019-07-25),   "user1",
    'rm',           datetime(2019-07-23),   "user2",
    'pwd',          datetime(2019-07-25),   "user2",
]
| summarize commands_details = make_list(pack('command', command, 'command_time', command_time)) by user_id
| mv-apply command_details = commands_details on
(
    order by todatetime(command_details['command_time']) asc
    | summarize make_list(tostring(command_details['command']))
)
| project-away commands_details
```

|`user_id`|`list_command_details_command`|
|---|---|
|user1|[<br>  "ls",<br>  "mkdir",<br>  "chmod",<br>  "dir",<br>  "pwd",<br>  "rm"<br>]|
|user2|[<br>  "rm",<br>  "pwd"<br>]|


**另请参阅**

* [mv-expand](./mvexpandoperator.md) 运算符。
