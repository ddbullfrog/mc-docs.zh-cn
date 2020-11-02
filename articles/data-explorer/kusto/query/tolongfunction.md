---
title: tolong() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 tolong()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: dfe05b9b4a30bdc1412d1920f7995f6185ceece6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104186"
---
# <a name="tolong"></a>tolong()

将输入转换为 long（带符号的 64 位）数字表示形式。

```kusto
tolong("123") == 123
```

> [!NOTE]
> 尽可能首选使用 [long()](./scalar-data-types/long.md)。

## <a name="syntax"></a>语法

`tolong(`Expr`)`

## <a name="arguments"></a>参数

* Expr：将转换为 long 的表达式。 

## <a name="returns"></a>返回

如果转换成功，则结果将是一个 long 类型的数字。
如果转换未成功，结果将是 `null`。
 
