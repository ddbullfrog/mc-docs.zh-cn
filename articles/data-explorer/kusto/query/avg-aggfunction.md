---
title: avg()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 avg()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 09/26/2019
ms.date: 10/29/2020
ms.openlocfilehash: 382c70940d4baf09c6eaeb843bda7a3d18f93c1e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104217"
---
# <a name="avg-aggregation-function"></a>avg()（聚合函数）

计算整组中 Expr 的平均值。 

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `avg(`Expr`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 具有 `null` 值的记录会被忽略，并且不会包含在计算中。

## <a name="returns"></a>返回

整组中 Expr 的平均值。
 