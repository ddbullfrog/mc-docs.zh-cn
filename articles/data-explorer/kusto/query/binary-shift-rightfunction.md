---
title: binary_shift_right() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 binary_shift_right()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 32f721de290748a3caa465c08eb922f96a96fe67
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106050"
---
# <a name="binary_shift_right"></a>binary_shift_right()

返回针对一对数字的二进制右移运算。

```kusto
binary_shift_right(x,y) 
```

## <a name="syntax"></a>语法

`binary_shift_right(`num1`,` num2 `)` 

## <a name="arguments"></a>参数

* num1、num2：长整型数字 。

## <a name="returns"></a>返回

返回针对一对数字的二进制右移运算：num1 >> (num2%64)。
如果 n 为负，则返回 NULL 值。