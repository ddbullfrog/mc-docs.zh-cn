---
title: sumif()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 sumif()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: c9956ab1c5c58fca4f348275e5c144ddf90ffcae
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105353"
---
# <a name="sumif-aggregation-function"></a>sumif()（聚合函数）

返回谓词计算结果为 `true` 的“Expr”的总和。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

还可以使用 [sum()](sum-aggfunction.md) 函数，该函数在没有谓词表达式的情况下对行求和。

## <a name="syntax"></a>语法

summarize `sumif(`*Expr*`,`*Predicate*`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 
* Predicate：谓词，如果为 true，则 Expr 的计算值将添加到总和。 

## <a name="returns"></a>返回

返回谓词计算结果为 `true` 的 Expr 总和值。

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
| summarize sumif(day_of_birth, strlen(name) > 4)
```

|sumif_day_of_birth|
|----|
|32|