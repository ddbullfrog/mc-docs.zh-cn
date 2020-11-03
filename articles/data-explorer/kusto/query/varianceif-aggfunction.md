---
title: varianceif()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 varianceif()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: c6c2bb7415c77953a1500ce1baef20ff681df7db
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106165"
---
# <a name="varianceif-aggregation-function"></a>Varianceif()（聚合函数）

计算“谓词”求值为 `true` 的组内的“Expr”的[变型](variance-aggfunction.md) 。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `varianceif(`*Expr*`, `*Predicate*`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 
* “谓词”：谓词如果为 true，则 Expr 计算值随即添加到变型中 。

## <a name="returns"></a>返回

谓词计算结果为 `true` 的组内 Expr 的变型值 。
 
## <a name="examples"></a>示例

```kusto
range x from 1 to 100 step 1
| summarize varianceif(x, x%2 == 0)

```

|varianceif_x|
|---|
|850|