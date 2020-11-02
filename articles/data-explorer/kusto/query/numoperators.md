---
title: 数值运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的数值运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: feac6f9efaa0569dafb586d5016942014fe0ae4d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103852"
---
# <a name="numerical-operators"></a>数值运算符

类型 `int`、`long` 和 `real` 表示数值类型。
可以在这些类型对之间使用以下运算符：

运算符       |描述                         |示例
---------------|------------------------------------|-----------------------
`+`            |添加                                 |`3.14 + 3.14`, `ago(5m) + 5m`
`-`            |减                            |`0.23 - 0.22`,
`*`            |相乘                            |`1s * 5`, `2 * 2`
`/`            |除                              |`10m / 1s`, `4 / 2`
`%`            |取模                              |`4 % 2`
`<`            |Less                                |`1 < 10`, `10sec < 1h`, `now() < datetime(2100-01-01)`
`>`            |Greater                             |`0.23 > 0.22`, `10min > 1sec`, `now() > ago(1d)`
`==`           |等于                              |`1 == 1`
`!=`           |不等于                          |`1 != 0`
`<=`           |Less or Equal                       |`4 <= 5`
`>=`           |Greater or Equal                    |`5 >= 4`
`in`           |等于某个元素       |[请参阅此处](inoperator.md)
`!in`          |不等于任何元素   |[请参阅此处](inoperator.md)

**与模数运算符有关的注释**

两个数字的取模始终在 Kusto 中返回一个小的非负数。
因此，两个数字的取模（“N” % “D”）如下 ：0 &le; ( *N* % *D* ) &lt; abs( *D* ).

例如，以下查询：

```kusto
print plusPlus = 14 % 12, minusPlus = -14 % 12, plusMinus = 14 % -12, minusMinus = -14 % -12
```

生成以下结果：

|plusPlus  | minusPlus  | plusMinus  | minusMinus|
|----------|------------|------------|-----------|
|2         | 10         | 2          | 10        |