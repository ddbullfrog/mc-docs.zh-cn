---
title: stdevif()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 stdevif()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 62469fe11a8b2d473e7cbeec7b1edf2c194462cb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105094"
---
# <a name="stdevif-aggregation-function"></a>stdevif()（聚合函数）

计算其谓词的计算结果为 `true` 的组内 Expr 的[stdev](stdev-aggfunction.md) 。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `stdevif(`*Expr*`, `*Predicate*`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 
* Predicate：谓词，如果为 true，则 Expr 计算值将添加到标准偏差中。

## <a name="returns"></a>返回

其谓词计算结果为 `true` 的组内 Expr 的标准偏差值 。
 
## <a name="examples"></a>示例

```kusto
range x from 1 to 100 step 1
| summarize stdevif(x, x%2 == 0)

```

|stdevif_x|
|---|
|29.1547594742265|