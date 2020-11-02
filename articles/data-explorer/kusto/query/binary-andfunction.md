---
title: binary_and() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 binary_and()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: eaa9cd60bc234a57f5c6125da096f091992fe021
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104952"
---
# <a name="binary_and"></a>binary_and()

返回两个值的“按位 `and`”运算的结果。

```kusto
binary_and(x,y) 
```

## <a name="syntax"></a>语法

`binary_and(`*num1*`,` *num2* `)`

## <a name="arguments"></a>参数

* num1、num2：长整型数字 。

## <a name="returns"></a>返回

返回对一对数字（num1 和 num2）的逻辑 AND 运算。