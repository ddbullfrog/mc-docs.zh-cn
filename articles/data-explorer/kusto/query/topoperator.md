---
title: top 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 top 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: fc6777eeac20ddaeb454e2503dfdbe1944ffb05c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105309"
---
# <a name="top-operator"></a>top 运算符

返回按指定列排序的前 *N* 列。

```kusto
T | top 5 by Name desc nulls last
```

## <a name="syntax"></a>语法

*T* `| top` *NumberOfRows* `by` *Expression* [`asc` | `desc`] [`nulls first` | `nulls last`]

## <a name="arguments"></a>参数

* NumberOfRows：要返回的 T 的行数。 可以指定任何数值表达式。
* *表达式* ：要作为排序依据的标量表达式。 值的类型必须是数字、日期、时间或字符串。
* `asc` 或 `desc`（默认）可能会控制实际从范围“底部”还是“顶部”进行选择。
* `nulls first`（`asc` 顺序的默认值）或 `nulls last`（`desc` 顺序的默认值）可以用来控制 null 值是在范围的开头还是结尾。

> [!TIP]
> `top 5 by name` 等效于语义和性能透视中的 `sort by name | take 5` 表达式。

## <a name="see-also"></a>请参阅 

* 使用 [top-nested](topnestedoperator.md) 运算符可生成分层的（嵌套的）排名靠前的结果。