---
title: binary_not() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 binary_not()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 8cd9d0bbdad1ba5790e9a6d758edccf48c8da7cc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104032"
---
# <a name="binary_not"></a>binary_not()

返回输入值的“按位求反”结果。

```kusto
binary_not(x)
```

## <a name="syntax"></a>语法

`binary_not(`*num1*`)`

## <a name="arguments"></a>参数

* num1：数字 

## <a name="returns"></a>返回

返回对一个数字 (num1) 的逻辑 NOT 运算。