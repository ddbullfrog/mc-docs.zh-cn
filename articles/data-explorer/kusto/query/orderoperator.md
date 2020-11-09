---
title: order 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 order 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 6825aad6bc1fa7d44ba69a0b9e01aa93ca52bd5f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103848"
---
# <a name="order-operator"></a>order 运算符 

按照一个或多个列的顺序对输入表的行排序。

```kusto
T | order by country asc, price desc
```

> [!NOTE]
> order 运算符是 sort 运算符的别名。 有关详细信息，请参阅 [sort 运算符](sortoperator.md)。

## <a name="syntax"></a>语法

*T* `| order by` *column* [`asc` | `desc`] [`nulls first` | `nulls last`] [`,` ...]

## <a name="arguments"></a>参数

* *T* ：要排序的表输入。
* column：T 的列，用作排序依据。 值的类型必须是数字、日期、时间或字符串。
* `asc` 按升序（即由低到高）排列。 默认值是 `desc`，降序，由高到低。
* `nulls first`（`asc` 顺序的默认值）将把 null 值放在开头，`nulls last`（`desc` 顺序的默认值）将把 null 值放在末尾。

