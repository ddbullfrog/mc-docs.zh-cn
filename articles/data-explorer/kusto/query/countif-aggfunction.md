---
title: countif()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 countif()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 08/02/2020
ms.date: 10/29/2020
ms.openlocfilehash: 0a09600f7acaf09f7362b0c0b385c064e86e3583
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104453"
---
# <a name="countif-aggregation-function"></a>countif()（聚合函数）

返回 *Predicate* 对其的计算结果为 `true` 的行数。 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用。

## <a name="syntax"></a>语法

汇总 `countif(`*Predicate*`)`

## <a name="arguments"></a>参数

*谓词* ：用于聚合计算的表达式。 Predicate 可以是返回类型为 bool 的任何标量表达式（计算结果为 true/false）。

## <a name="returns"></a>返回

返回 *Predicate* 对其的计算结果为 `true` 的行数。

## <a name="example"></a>示例

```kusto
let T = datatable(name:string, day_of_birth:long)
[
   "John", 9,
   "Paul", 18,
   "George", 25,
   "Ringo", 7
];
T
| summarize countif(strlen(name) > 4)
```

|countif_|
|----|
|2|

## <a name="see-also"></a>请参阅

[count()](count-aggfunction.md) 函数，用于在没有谓词表达式的情况下计算行数。
