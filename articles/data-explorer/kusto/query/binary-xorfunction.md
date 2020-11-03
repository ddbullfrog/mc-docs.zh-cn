---
title: Binary_xor() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 binary_xor()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 17d7db0aa5e5ea7d8c15d4cdb4e3e98ce6a921e4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106049"
---
# <a name="binary_xor"></a>binary_xor()

返回两个值的位 `xor` 运算的结果。

```kusto
binary_xor(x,y)
```

## <a name="syntax"></a>语法

`binary_xor(`*num1*`,` *num2* `)`

## <a name="arguments"></a>参数

* num1、num2：长整型数字 。

## <a name="returns"></a>返回

返回对一对数字的逻辑 XOR 运算：num1 ^ num2。