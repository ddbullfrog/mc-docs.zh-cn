---
title: make_bag_if()（聚合函数）- Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 make_bag_if()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 422fd456ea65d5977fd709d75a551f1e15ef22a9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105593"
---
# <a name="make_bag_if-aggregation-function"></a>make_bag_if()（聚合函数）

返回组中“Expr”的所有值的 `dynamic` (JSON) 属性包（字典），其 Predicate 的计算结果为 `true`。

> [!NOTE]
> 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用。

## <a name="syntax"></a>语法

`summarize` `make_bag_if(`*Expr* , *Predicate* [`,` *MaxSize* ]`)`

## <a name="arguments"></a>参数

* Expr：将用于聚合计算的 `dynamic` 类型的表达式。
* *谓词* ：必须计算为 `true` 的谓词，用于将“Expr”添加到结果中。
* MaxSize：对返回元素最大数目的可选整数限制（默认值是 1048576）。 MaxSize 值不能超过 1048576。

## <a name="returns"></a>返回

返回组中“Expr”的所有值（属性包（字典））的 `dynamic` (JSON) 属性包（字典），其 Predicate 的计算结果为 `true`。
将跳过非字典值。
如果一个键出现在多个行中，则会从此键的可能值中选择一个任意值。

> [!NOTE]
> [`make_bag`](./make-bag-aggfunction.md) 函数类似于不带谓词表达式的 make_bag_if()。

## <a name="examples"></a>示例

```kusto
let T = datatable(prop:string, value:string, predicate:bool)
[
    "prop01", "val_a", true,
    "prop02", "val_b", false,
    "prop03", "val_c", true
];
T
| extend p = pack(prop, value)
| summarize dict=make_bag_if(p, predicate)

```

|dict|
|----|
|{ "prop01": "val_a", "prop03": "val_c" } |

使用 [bag_unpack()](bag-unpackplugin.md) 插件将 make_bag_if() 输出中的包键转换为列。 

```kusto
let T = datatable(prop:string, value:string, predicate:bool)
[
    "prop01", "val_a", true,
    "prop02", "val_b", false,
    "prop03", "val_c", true
];
T
| extend p = pack(prop, value)
| summarize bag=make_bag_if(p, predicate)
| evaluate bag_unpack(bag)

```

|prop01|prop03|
|---|---|
|val_a|val_c|
