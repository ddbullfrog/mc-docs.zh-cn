---
title: make_list_with_nulls()（聚合函数） - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 make_list_with_nulls()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/09/2020
ms.date: 09/24/2020
ms.openlocfilehash: cebd1f0bc5b859dde8e042e00817844ee34c9dea
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146532"
---
# <a name="make_list_with_nulls-aggregation-function"></a>make_list_with_nulls()（聚合函数）

返回组中 Expr 的所有值的 `dynamic` (JSON) 数组（包括 null 值）。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

`summarize` `make_list_with_nulls(` *Expr* `)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。

## <a name="returns"></a>返回

返回组中 Expr 的所有值的 `dynamic` (JSON) 数组（包括 null 值）。
如果未对 `summarize` 运算符的输入进行排序，则生成的数组中的元素顺序是不确定的。
如果对 `summarize` 运算符的输入进行了排序，则生成的数组中的元素顺序和输入一样。

> [!TIP]
> 使用 [`mv-apply`](./mv-applyoperator.md) 运算符按某个键值创建一个有序列表。 请参阅[此处的](./mv-applyoperator.md#using-the-mv-apply-operator-to-sort-the-output-of-make_list-aggregate-by-some-key)示例。
