---
title: tdigest_merge()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 tdigest_merge()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/09/2019
ms.date: 09/30/2020
ms.openlocfilehash: e8d9b23b17f13aee91e2cd34fd2ac2c448526e25
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103987"
---
# <a name="tdigest_merge-aggregation-function"></a>tdigest_merge()（聚合函数）

将 tdigest 结果合并到组中。 

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用。

在[此处](percentiles-aggfunction.md#estimation-error-in-percentiles)详细了解基础算法 (T-Digest) 和预估误差。

## <a name="syntax"></a>语法

summarize `tdigest_merge(`*Expr*`)`。

summarize `tdigest_merge(`*Expr*`)` - 别名。

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 

## <a name="returns"></a>返回

组中 Expr 的合并 tdigest 值。
 

**提示**

1) 可以使用函数 [`percentile_tdigest()`] (percentile-tdigestfunction.md)。

2) 同一组中包含的所有 tdigest 必须属于同一类型。