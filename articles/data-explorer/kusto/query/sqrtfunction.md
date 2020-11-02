---
title: sqrt() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 sqrt()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 27e356220d424dafcd4195df0fd585c90f356984
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103569"
---
# <a name="sqrt"></a>sqrt()

返回平方根函数。  

## <a name="syntax"></a>语法

`sqrt(`*x*`)`

## <a name="arguments"></a>参数

* x：大于或等于 0 的实数。

## <a name="returns"></a>返回

* 正数值，例如 `sqrt(x) * sqrt(x) == x`
* 如果参数为负或不能转换为 `real` 值，则为 `null`。 