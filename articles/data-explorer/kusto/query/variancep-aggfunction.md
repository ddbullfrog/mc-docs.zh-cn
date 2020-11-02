---
title: variancep()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 variancep()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: f6d57d1c8eda74cacd5e5c391fdcf996e7b6247e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104384"
---
# <a name="variancep-aggregation-function"></a>variancep()（聚合函数）

将某组视为总体，计算该组内 Expr 的方差。 

* 使用的公式：

:::image type="content" source="images/variancep-aggfunction/variance-population.png" alt-text="方差总体":::

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `variancep(`Expr`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 

## <a name="returns"></a>返回

组中 Expr 的方差值。
 
## <a name="examples"></a>示例

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), variancep(x) 
```

|list_x|variance_x|
|---|---|
|[ 1, 2, 3, 4, 5]|2|