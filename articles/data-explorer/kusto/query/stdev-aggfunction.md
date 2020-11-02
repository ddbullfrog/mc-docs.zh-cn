---
title: stdev()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 stdev()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 24a710ecea8f77dd2be2979358a709fe87146182
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105096"
---
# <a name="stdev-aggregation-function"></a>stdev()（聚合函数）

将该组作为示例，计算组内 Expr 的标准偏差。 

* 使用的公式：

:::image type="content" source="images/stdev-aggfunction/stdev-sample.png" alt-text="Stdev 示例":::

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `stdev(`Expr`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 

## <a name="returns"></a>返回

组内 Expr 的标准偏差值。
 
## <a name="examples"></a>示例

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), stdev(x)

```

|list_x|stdev_x|
|---|---|
|[ 1, 2, 3, 4, 5]|1.58113883008419|