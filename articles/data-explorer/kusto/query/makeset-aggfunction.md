---
title: make_set()（聚合函数） - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 make_set()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 01/23/2020
ms.date: 10/29/2020
ms.openlocfilehash: 50593b2d57b66183e21877c3e5c4203d00ab96af
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105781"
---
# <a name="make_set-aggregation-function"></a>make_set()（聚合函数）

返回 *Expr* 在组中所获取非重复值集的 `dynamic` (JSON) 数组。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

`summarize` `make_set(`*Expr* [`,` *MaxSize* ]`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。
* MaxSize 是对返回元素最大数目的可选整数限制（默认值是 1048576）。 MaxSize 值不能超过 1048576。

> [!NOTE]
> 函数 `makeset()` 的旧版和已过时变体的默认限制为 MaxSize = 128。

## <a name="returns"></a>返回

返回 *Expr* 在组中所获取非重复值集的 `dynamic` (JSON) 数组。
数组的排序顺序未定义。

> [!TIP]
> 若要仅对非重复值进行计数，请使用 [dcount()](dcount-aggfunction.md)

## <a name="example"></a>示例

```kusto
PageViewLog 
| summarize countries=make_set(country) by continent
```

:::image type="content" source="images/makeset-aggfunction/makeset.png" alt-text="显示 Kusto 查询的表在Azure 数据资源管理器中按洲汇总了国家/地区":::

## <a name="see-also"></a>请参阅

* 将 [`mv-expand`](./mvexpandoperator.md) 运算符用于相反的函数。
* [`make_set_if`](./makesetif-aggfunction.md) 运算符与 `make_set` 相似，只是它也接受谓词。