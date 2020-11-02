---
title: todecimal() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 todecimal()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: f4258fab435b667685e0275c782b69f852fc926e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103978"
---
# <a name="todecimal"></a>todecimal()

将输入转换为十进制数表示形式。

```kusto
todecimal("123.45678") == decimal(123.45678)
```

## <a name="syntax"></a>语法

`todecimal(`*Expr*`)`

## <a name="arguments"></a>参数

* Expr：将转换为十进制的表达式。 

## <a name="returns"></a>返回

如果转换成功，则结果将为十进制数。
如果转换未成功，结果将是 `null`。
 
*注意* ：尽可能首选使用 [real()](./scalar-data-types/real.md)。