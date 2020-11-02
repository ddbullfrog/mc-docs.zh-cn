---
title: hll_merge()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 hll_merge()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/15/2019
ms.date: 10/29/2020
ms.openlocfilehash: e36549fbeb6fa2e7faaea80e2d87b85504314d31
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103887"
---
# <a name="hll_merge-aggregation-function"></a>hll_merge()（聚合函数）

将组中的 `HLL` 结果合并为单个 `HLL` 值。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用。

有关详细信息，请参阅[基础算法 (HyperLogLog  ) 和估算准确度](dcount-aggfunction.md#estimation-accuracy)。

## <a name="syntax"></a>语法

`summarize` `hll_merge(`*Expr*`)`

## <a name="arguments"></a>参数

* `*Expr*`：将要用于聚合计算的表达式。

## <a name="returns"></a>返回

此函数返回组中 `*Expr*` 的合并 `hll` 值。
 
**提示**

1) 使用函数 [dcount_hll] (dcount-hllfunction.md) 根据 `hll` / `hll-merge` 聚合函数计算 `dcount`。
