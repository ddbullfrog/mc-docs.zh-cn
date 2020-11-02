---
title: arg_min()（聚合函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 arg_min()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/12/2019
ms.date: 10/29/2020
ms.openlocfilehash: 01a905cc3a45ec6e61efe9f5b051119ed925ae7c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105052"
---
# <a name="arg_min-aggregation-function"></a>arg_min()（聚合函数）

在最小化 ExprToMinimize 的组中查找行，并返回 ExprToReturn 的值（或使用 `*` 返回整个行） 。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

`summarize` [`(`NameExprToMinimize `,` NameExprToReturn [`,` ...] `)=`] `arg_min` `(`ExprToMinimize, `*` | ExprToReturn  [`,` ...]`)`   

## <a name="arguments"></a>参数

* ExprToMinimize：用于聚合计算的表达式。 
* ExprToReturn：当 ExprToMinimize 为最小值时，用于返回值的表达式。 要返回的表达式可以是通配符 (*)，用于返回输入表的所有列。
* NameExprToMinimize：表示 ExprToMinimize 的结果列的可选名称。
* NameExprToReturn：表示 ExprToReturn 的结果列的其他可选名称。

## <a name="returns"></a>返回

在最小化 ExprToMinimize 的组中查找行，并返回 ExprToReturn 的值（或使用 `*` 返回整个行） 。

## <a name="examples"></a>示例

显示每个产品的最低价供应商：

```kusto
Supplies | summarize arg_min(Price, Supplier) by Product
```

显示所有详细信息，而不仅仅是供应商名称：

```kusto
Supplies | summarize arg_min(Price, *) by Product
```

找到每个大洲最南端的城市及其所属的国家/地区：

```kusto
PageViewLog 
| summarize (latitude, min_lat_City, min_lat_country)=arg_min(latitude, City, country) 
    by continent
```

:::image type="content" source="images/arg-min-aggfunction/arg-min.png" alt-text="Arg min":::
