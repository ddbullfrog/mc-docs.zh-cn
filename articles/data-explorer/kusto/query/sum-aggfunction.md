---
title: sum()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 sum()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 556dd79a9875c14c36f5816022c00725db77041c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104382"
---
# <a name="sum-aggregation-function"></a>sum()（聚合函数）

计算组中 Expr 的总和。 

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `sum(`Expr`)`

## <a name="arguments"></a>参数

* Expr：用于聚合计算的表达式。 

## <a name="returns"></a>返回

组中 Expr 的总和值。
 