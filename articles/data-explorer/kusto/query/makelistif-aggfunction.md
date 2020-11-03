---
title: make_list_if()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 make_list_if()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 272df1d1a4a34ae1897a8949b62e5ca07ef52657
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105819"
---
# <a name="make_list_if-aggregation-function"></a>make_list_if()（聚合函数）

返回组中“Expr”的所有值的 `dynamic` (JSON) 数组，其 Predicate 的计算结果为 `true`。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

`summarize` `make_list_if(`*Expr* , *Predicate* [`,` *MaxSize* ]`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。
* *谓词* ：必须计算为 `true` 的谓词，用于将“Expr”添加到结果中。
* MaxSize 是对返回元素最大数目的可选整数限制（默认值是 1048576）。 MaxSize 值不能超过 1048576。

## <a name="returns"></a>返回

返回组中“Expr”的所有值的 `dynamic` (JSON) 数组，其 Predicate 的计算结果为 `true`。
如果未对 `summarize` 运算符的输入进行排序，那么生成的数组中的元素顺序是不确定的。
如果对 `summarize` 运算符的输入进行了排序，则生成的数组中的元素顺序和输入一样。

## <a name="example"></a>示例

```kusto
let T = datatable(name:string, day_of_birth:long)
[
   "John", 9,
   "Paul", 18,
   "George", 25,
   "Ringo", 7
];
T
| summarize make_list_if(name, strlen(name) > 4)
```

|list_name|
|----|
|["George", "Ringo"]|

## <a name="see-also"></a>请参阅

[`make_list`](./makelist-aggfunction.md) 函数，它在无谓词表达式的情况下执行相同的操作。