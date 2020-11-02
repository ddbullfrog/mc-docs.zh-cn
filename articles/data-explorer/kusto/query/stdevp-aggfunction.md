---
title: stdevp()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 stdevp()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: df5fda760a351ddb2a933a422aa4d972cdc84977
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104007"
---
# <a name="stdevp-aggregation-function"></a>stdevp()（聚合函数）

将组视为一个总体，计算整个组中 Expr 的标准偏差。 

* 使用的公式：

:::image type="content" source="images/stdevp-aggfunction/stdev-population.png" alt-text="Stdev 总体":::

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `stdevp(`Expr`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 

## <a name="returns"></a>返回

整个组内 Expr 的标准偏差值。
 
## <a name="examples"></a>示例

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), stdevp(x)

```

|list_x|stdevp_x|
|---|---|
|[ 1, 2, 3, 4, 5]|1.4142135623731|