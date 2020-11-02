---
title: lookup 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 lookup 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/12/2020
ms.date: 10/29/2020
ms.openlocfilehash: f6f2ee6e964d509bb36ffd90ffd312d2f1a84fa6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105595"
---
# <a name="lookup-operator"></a>lookup 运算符

`lookup` 运算符使用在维度表中查找的值扩展事实数据表的列。

```kusto
FactTable | lookup kind=leftouter (DimensionTable) on CommonColumn, $left.Col1 == $right.Col2
```

此处，结果是一个表，它根据后一个表中的每对 (`CommonColumn1`,`Col2`) 查找前一个表中的每对 (`CommonColumn`,`Col`)，使用来自 `DimensionTable`（通过 `$right` 进行引用）的数据来扩展 `FactTable` (`$left`)。 有关事实数据表与维度表之间的差异，请参阅[事实数据表与维度表](../concepts/fact-and-dimension-tables.md)。 

`lookup` 运算符执行的操作类似于 [join 运算符](joinoperator.md)，但存在以下差异：

* 结果不会重复 `$right` 表中作为联接操作基础的列。
* 仅支持两种查找：`leftouter` 和 `inner`，并且 `leftouter` 为默认值。
* 从性能方面考虑，系统默认情况下假定 `$left` 表是较大的（事实数据）表，而 `$right` 表是较小的（维度）表。 这与 `join` 运算符使用的假设完全相反。
* `lookup` 运算符自动将 `$right` 表广播到 `$left` 表（实质上，其行为与指定了 `hint.broadcast` 时的行为相同）。 请注意，这会限制 `$right` 表的大小。

## <a name="syntax"></a>语法

*LeftTable* `|` `lookup` [`kind` `=` (`leftouter`|`inner`)] `(` *RightTable* `)` `on` *Attributes*

## <a name="arguments"></a>参数

* LeftTable：用作查找基础的表或表格表达式。
  表示为 `$left`。

* RightTable：用于“填充”事实数据表中的新列的表或表格表达式。 表示为 `$right`。

* Attributes：一个或多个规则的逗号分隔列表，这些规则描述 *LeftTable* 中的行如何与 *RightTable* 中的行进行匹配。 将使用 `and` 逻辑运算符评估多个规则。
  规则可以是下列项之一：

  |规则类型        |语法                                          |Predicate                                                      |
  |-----------------|------------------------------------------------|---------------------------------------------------------------|
  |基于名称的等式 |*ColumnName*                                    |`where` *LeftTable* . *ColumnName* `==` *RightTable* . *ColumnName*|
  |基于值的等式|`$left.`*LeftColumn* `==` `$right.`*RightColumn*|`where` `$left.`*LeftColumn* `==` `$right.`*RightColumn        |

  > [!Note] 
  > 如果使用“基于值的等式”，则列名称必须通过由 `$left` 和 `$right` 表示法表示的相应所有者表进行限定。

* `kind`：一个可选说明，指示如何处理 LeftTable 中在 RightTable 中没有匹配项的行。 默认情况下将使用 `leftouter`，这意味着所有这些行都将出现在输出中，对于此运算符添加的 RightTable 列的缺失值，将使用 null 值。 如果使用 `inner`，则输出中将省略此类行。 （`looku`p 运算符不支持其他种类的联接。）
  
## <a name="returns"></a>返回

具有以下内容的表：

* 一列，用于每个表中的每一列，包括匹配键。
  如果存在名称冲突，会自动重命名右侧的列。
* 一行，用于输入表之间的所有匹配项。 匹配项是从表中选出的一行，该表中的所有 `on` 字段值与其他表中的值相同。 
* 属性（查找键）在输出表中将只出现一次。

 * `kind` 未指定，`kind=leftouter`

     除了内部匹配，还有一行用于左侧（和/或右侧）的每一行（即使没有匹配项）。 在这种情况下，不匹配的输出单元格包含 null。

 * `kind=inner`

     输出中存在一行，用于左侧和右侧匹配行的每个组合。

## <a name="examples"></a>示例

```kusto
let FactTable=datatable(Row:string,Personal:string,Family:string) [
  "1", "Bill",   "Gates",
  "2", "Bill",   "Clinton",
  "3", "Bill",   "Clinton",
  "4", "Steve",  "Ballmer",
  "5", "Tim",    "Cook"
];
let DimTable=datatable(Personal:string,Family:string,Alias:string) [
  "Bill",  "Gates",   "billg",
  "Bill",  "Clinton", "billc",
  "Steve", "Ballmer", "steveb",
  "Tim",   "Cook",    "timc"
];
FactTable
| lookup kind=leftouter DimTable on Personal, Family
```

行     | 个人  | 系列   | Alias
--------|-----------|----------|--------
1       | Bill      | Gates    | billg
2       | Bill      | Clinton  | billc
3       | Bill      | Clinton  | billc
4       | Steve     | Ballmer  | steveb
5       | Tim       | Cook     | timc