---
title: binary_shift_left() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 binary_shift_left()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4e56b5a0e7a42897172d8a4f12dea0770ce2821e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106051"
---
# <a name="binary_shift_left"></a>binary_shift_left()

返回针对一对数字的二进制左移运算。

```kusto
binary_shift_left(x,y)  
```

## <a name="syntax"></a>语法

`binary_shift_left(`num1`,` num2 `)` 

## <a name="arguments"></a>参数

* num1、num2：int 数字 。

## <a name="returns"></a>返回

返回针对一对数字的二进制左移运算：num1 << (num2%64)。
如果 n 为负，则返回 NULL 值。