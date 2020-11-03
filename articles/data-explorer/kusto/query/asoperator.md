---
title: as 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 as 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 05724427849c1085dff1f9bf973022f0dddcf9f0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106310"
---
# <a name="as-operator"></a>as 运算符

将名称绑定到运算符的输入表格表达式，从而允许查询多次引用表格表达式的值，而不会中断查询，并通过 [let 语句](letstatement.md)绑定名称。

## <a name="syntax"></a>语法

*T* `|` `as` [`hint.materialized` `=` `true`] *Name*

## <a name="arguments"></a>参数

* *T* ：表格表达式。
* *名称* ：表格表达式的临时名称。
* `hint.materialized`：如果设置为 `true`，则表格表达式的值将被具体化，就像它已由 [materialize()](./materializefunction.md) 函数调用包装了一样。

> [!NOTE]
> * `as` 提供的名称将用于 [union](./unionoperator.md) 的 `withsource=` 列、[find](./findoperator.md) 的 `source_` 列和 [search](./searchoperator.md) 的 `$table` 列。
> * 在 [join](./joinoperator.md) 的外部表格输入 (`$left`) 中使用运算符命名的表格表达式也可用于 join 的表格内部输入 (`$right`)。

## <a name="examples"></a>示例

```kusto
// 1. In the following 2 example the union's generated TableName column will consist of 'T1' and 'T2'
range x from 1 to 10 step 1 
| as T1 
| union withsource=TableName T2

union withsource=TableName (range x from 1 to 10 step 1 | as T1), T2

// 2. In the following example, the 'left side' of the join will be: 
//      MyLogTable filtered by type == "Event" and Name == "Start"
//    and the 'right side' of the join will be: 
//      MyLogTable filtered by type == "Event" and Name == "Stop"
MyLogTable  
| where type == "Event"
| as T
| where Name == "Start"
| join (
    T
    | where Name == "Stop"
) on ActivityId
```