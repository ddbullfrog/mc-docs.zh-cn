---
title: Project 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 Project 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 81856ffe264ce0ed135f88922bb27d27ce5fc540
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104193"
---
# <a name="project-operator"></a>project 运算符

选择要包含、重命名或删除的列，并插入新的计算列。 

结果中的列顺序由参数的顺序指定。 只有在参数中指定的列才包含在结果中。 输入中的任何其他列都会被删除。  （另请参阅 `extend`。）

```kusto
T | project cost=price*quantity, price
```

## <a name="syntax"></a>语法

*T* `| project` *ColumnName* [`=` *Expression* ] [`,` ...]
  
或
  
*T* `| project` [ *ColumnName* | `(`*ColumnName* [`,`]`)` `=`] *Expression* [`,` ...]

## <a name="arguments"></a>参数

* *T* ：输入表。
* ColumnName：要在输出中显示的列的可选名称。 如果没有 Expression，则 ColumnName 是必需的，输入中必须出现该名称的列。 如果省略该项，则会自动生成名称。 如果 Expression 返回多个列，则可在括号中指定列名列表。 在这种情况下，会将指定的名称赋予 Expression 的输出列，并会删除所有其余的输出列（如果有）。 如果未指定列名列表，则 Expression 的所有具有生成名称的输出列都将添加到输出中。
* *Expression* ：引用输入列的可选标量表达式。 如果未省略 ColumnName，则 Expression 是必需的。

    允许返回与输入中现有列具有相同名称的新计算列。

## <a name="returns"></a>返回

一个表，其中包含名为参数的列和与输入表中相同数量的行。

## <a name="example"></a>示例

以下示例演示可使用 `project` 运算符执行的几种操作。 输入表 `T` 具有属于 `int` 类型的三列：`A`、`B` 和 `C`。 

```kusto
T
| project
    X=C,                       // Rename column C to X
    A=2*B,                     // Calculate a new column A from the old B
    C=strcat("-",tostring(C)), // Calculate a new column C from the old C
    B=2*B                      // Calculate a new column B from the old B
```

[series_stats](series-statsfunction.md) 是返回多个列的函数的示例。