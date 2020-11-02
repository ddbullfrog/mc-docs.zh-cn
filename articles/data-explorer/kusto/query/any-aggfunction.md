---
title: Any()（聚合函数） - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 any()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 7cbe31bd0fa7ed170a8efa1d29ebdfffa19dc27c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105058"
---
# <a name="any-aggregation-function"></a>Any()（聚合函数）

任意为[汇总运算符](summarizeoperator.md)中的每个组选择一条记录，并返回对每个此类记录的一个或多个表达式的值。

## <a name="syntax"></a>语法

`summarize` `any` `(` ( *Expr* [`,` *Expr2* ...]) | `*` `)`

## <a name="arguments"></a>参数

* Expr：从要返回的输入中选择的每条记录的表达式。
* *Expr2* ： *ExprN* ：其他表达式。

## <a name="returns"></a>返回

`any` 聚合函数返回为每个记录计算的表达式的值，这些值是从每组汇总运算符中随机选择的。

如果提供了 `*` 参数，则该函数的行为就像表达式是汇总运算符禁止分组依据列（如果有）的输入的所有列一样。

**备注**

此函数有助于获取每个复合组键值的一列或多列的示例值。

当使用单个列引用提供函数时，该函数将尝试返回非 null/非空值（如果存在此类值）。

由于此函数的随机本质，在 `summarize` 运算符的单个应用中多次使用该函数并不等效于单次将其用于多个表达式。 前者可以让每个应用选择不同的记录，而后者可保证所有值都是通过单个记录（每个不同的组）计算得出的。

## <a name="examples"></a>示例

显示随机洲：

```kusto
Continents | summarize any(Continent)
```

:::image type="content" source="images/aggfunction/any1.png" alt-text="Any 1":::

显示随机记录的所有详细信息：

```kusto
Continents | summarize any(*)
```

:::image type="content" source="images/aggfunction/any2.png" alt-text="Any 1":::

显示每个随机洲的所有详细信息：

```kusto
Continents | summarize any(*) by Continent
```

:::image type="content" source="images/aggfunction/any3.png" alt-text="Any 1":::
