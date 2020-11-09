---
title: make_list_with_nulls()（聚合函数） - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 make_list_with_nulls()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/09/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1c2c81bc7e73d8724afad6927e78d33521c9b5f0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105591"
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
> 使用 [`array_sort_asc()`](./arraysortascfunction.md) 或 [`array_sort_desc()`](./arraysortdescfunction.md) 函数按某个键创建一个有序列表。
