---
title: serialize 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 serialize 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 018c6d51095bca266428e9ee62f4d5bac02c0832
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106189"
---
# <a name="serialize-operator"></a>serialize 运算符

标记输入行集的顺序可安全用于开窗函数。

运算符具有声明性含义。 它将输入行集标记为已序列化（已排序），以便可以将[开窗函数](./windowsfunctions.md)应用于它。

```kusto
T | serialize rn=row_number()
```

## <a name="syntax"></a>语法

`serialize` [Name1 `=` Expr1 [`,` Name2 `=` Expr2]...]   

* Name/Expr 对与 [extend 运算符](./extendoperator.md)中的对相似 。

## <a name="example"></a>示例

```kusto
Traces
| where ActivityId == "479671d99b7b"
| serialize

Traces
| where ActivityId == "479671d99b7b"
| serialize rn = row_number()
```

以下运算符的输出行集标记为已序列化。

[range](./rangeoperator.md)、[sort](./sortoperator.md)、[order](./orderoperator.md)、[top](./topoperator.md)、[top-hitters](./tophittersoperator.md)、[getschema](./getschemaoperator.md)。

以下运算符的输出行集标记为非序列化。

[sample](./sampleoperator.md)、[sample-distinct](./sampledistinctoperator.md)、[distinct](./distinctoperator.md)、[join](./joinoperator.md)、[top-nested](./topnestedoperator.md)、[count](./countoperator.md)、[summarize](./summarizeoperator.md)、[facet](./facetoperator.md)、[mv-expand](./mvexpandoperator.md)、[evaluate](./evaluateoperator.md)、[reduce by](./reduceoperator.md)、[make-series](./make-seriesoperator.md)

所有其他运算符保留序列化属性。 如果输入行集已序列化，则输出行集也会序列化。
