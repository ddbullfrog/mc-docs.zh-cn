---
title: todouble()/toreal() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 todouble()/toreal()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: caf53e3b0a6a4dd12dfd11f0219df0b23d61112b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103977"
---
# <a name="todouble-toreal"></a>todouble(), toreal()

将输入转换为 `real` 类型的值。 （`todouble()` 和 `toreal()` 是同义函数。）

```kusto
toreal("123.4") == 123.4
```

> [!NOTE]
> 如果可能，最好使用 [double() 或 real()](./scalar-data-types/real.md)。

## <a name="syntax"></a>语法

`toreal(`*Expr*`)`
`todouble(`*Expr*`)`

## <a name="arguments"></a>参数

* Expr：一个表达式，其值将转换为 `real` 类型的值。

## <a name="returns"></a>返回

如果转换成功，则结果为 `real` 类型的值。
如果转换不成功，则结果为 `real(null)` 值。
