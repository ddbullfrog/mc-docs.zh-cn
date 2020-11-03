---
title: min()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 min()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 07/24/2019
ms.date: 10/29/2020
ms.openlocfilehash: ffcc8a11553b9a510c6f997bd0ff424d0f26e0ba
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105991"
---
# <a name="min-aggregation-function"></a>min()（聚合函数）

返回组内的最小值。 

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

`summarize` `min(`*Expr*`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 

## <a name="returns"></a>返回

组内 Expr 的最小值。
 
> [!TIP]
> 这能提供该项本身的最大值或最小值 - 例如最高或最低价格。 但如果需要行中的其他列 - 例如，提供最低价格的供应商的名称 - 则需使用 [arg_max](arg-max-aggfunction.md) 或 [arg_min](arg-min-aggfunction.md)。