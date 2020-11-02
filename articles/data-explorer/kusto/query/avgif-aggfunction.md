---
title: avgif()（聚合函数） - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 avgif()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 2980da50003a31bfade49382f0de904c4bc96f9d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104216"
---
# <a name="avgif-aggregation-function"></a>avgif()（聚合函数）

计算谓词的计算结果为 `true` 的组内 Expr 的[平均值](avg-aggfunction.md) 。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `avgif(`*Expr*`, `*Predicate*`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 具有 `null` 值的记录将被忽略，并且不包括在计算中。
* *谓词* ：谓词如果为 true，则 Expr 计算值将添加到平均值。

## <a name="returns"></a>返回

谓词计算结果为 `true` 的组内 Expr 的平均值 。
 
## <a name="examples"></a>示例

```kusto
range x from 1 to 100 step 1
| summarize avgif(x, x%2 == 0)
```

|avgif_x|
|---|
|51|