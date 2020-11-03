---
title: asin() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 asin()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: 85fa309d9eaccb739b09a45e01924af5133f8dbb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106166"
---
# <a name="asin"></a>asin()

返回其正弦为指定数字的角度（[`sin()`](sinfunction.md) 的反运算）。

## <a name="syntax"></a>语法

`asin(`*x*`)`

## <a name="arguments"></a>参数

* x：范围 [-1, 1] 中的实数。

## <a name="returns"></a>返回

* `x` 的反正弦的值
* `null` 如果 `x` < -1 或者 `x` > 1