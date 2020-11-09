---
title: make_list()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 make_list()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 01/23/2020
ms.date: 10/29/2020
ms.openlocfilehash: e5a1c9a76a2ff886468cc32e4864670aca4d52f0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104609"
---
# <a name="make_list-aggregation-function"></a>make_list()（聚合函数）

返回组中 *Expr* 所有值的 `dynamic` (JSON) 数组。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

`summarize` `make_list(`*Expr* [`,` *MaxSize* ]`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。
* MaxSize 是对返回元素最大数目的可选整数限制（默认值是 1048576）。 MaxSize 值不能超过 1048576。

> [!NOTE]
> 函数 `makelist()` 的旧版和已过时变体的默认限制为 MaxSize = 128。

## <a name="returns"></a>返回

返回组中 *Expr* 所有值的 `dynamic` (JSON) 数组。
如果未对 `summarize` 运算符的输入进行排序，那么生成的数组中的元素顺序是不确定的。
如果对 `summarize` 运算符的输入进行了排序，则生成的数组中的元素顺序和输入一样。

> [!TIP]
> 使用 [`array_sort_asc()`](./arraysortascfunction.md) 或 [`array_sort_desc()`](./arraysortdescfunction.md) 函数按某个键创建一个有序列表。

## <a name="examples"></a>示例

### <a name="one-column"></a>一列

最简单的示例是基于一列生成一个列表：

```kusto
let shapes = datatable (name: string, sideCount: int)
[
    "triangle", 3,
    "square", 4,
    "rectangle", 4,
    "pentagon", 5,
    "hexagon", 6,
    "heptagon", 7,
    "octogon", 8,
    "nonagon", 9,
    "decagon", 10
];
shapes
| summarize mylist = make_list(name)
```

|mylist|
|---|
|["triangle","square","rectangle","pentagon","hexagon","heptagon","octogon","nonagon","decagon"]|

### <a name="using-the-by-clause"></a>使用“by”子句

在下面的查询中，使用了 `by` 子句进行分组：

```kusto
let shapes = datatable (name: string, sideCount: int)
[
    "triangle", 3,
    "square", 4,
    "rectangle", 4,
    "pentagon", 5,
    "hexagon", 6,
    "heptagon", 7,
    "octogon", 8,
    "nonagon", 9,
    "decagon", 10
];
shapes
| summarize mylist = make_list(name) by isEvenSideCount = sideCount % 2 == 0
```

|mylist|isEvenSideCount|
|---|---|
|false|["triangle","pentagon","heptagon","nonagon"]|
|是|["square","rectangle","hexagon","octogon","decagon"]|

### <a name="packing-a-dynamic-object"></a>将动态对象打包

可以在列中将一个动态对象[打包](./packfunction.md)，然后基于其生成一个列表，如以下查询所示：

```kusto
let shapes = datatable (name: string, sideCount: int)
[
    "triangle", 3,
    "square", 4,
    "rectangle", 4,
    "pentagon", 5,
    "hexagon", 6,
    "heptagon", 7,
    "octogon", 8,
    "nonagon", 9,
    "decagon", 10
];
shapes
| extend d = pack("name", name, "sideCount", sideCount)
| summarize mylist = make_list(d) by isEvenSideCount = sideCount % 2 == 0
```

|mylist|isEvenSideCount|
|---|---|
|false|[{"name":"triangle","sideCount":3},{"name":"pentagon","sideCount":5},{"name":"heptagon","sideCount":7},{"name":"nonagon","sideCount":9}]|
|是|[{"name":"square","sideCount":4},{"name":"rectangle","sideCount":4},{"name":"hexagon","sideCount":6},{"name":"octogon","sideCount":8},{"name":"decagon","sideCount":10}]|

## <a name="see-also"></a>另请参阅

[`make_list_if`](./makelistif-aggfunction.md) 运算符与 `make_list` 相似，只是它也接受谓词。