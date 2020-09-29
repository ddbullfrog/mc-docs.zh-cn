---
title: Buildschema()（聚合函数）- Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 buildschema()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/24/2020
ms.openlocfilehash: b0cf8350939f508b86b69bc2ec949c2d295a4f33
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146762"
---
# <a name="buildschema-aggregation-function"></a>Buildschema()（聚合函数）

返回允许“DynamicExpr”所有值的最小架构。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `buildschema(`*DynamicExpr*`)`

## <a name="arguments"></a>参数

* *DynamicExpr*：用于聚合计算的表达式。 参数列类型必须为 `dynamic`。 

## <a name="returns"></a>返回

组内“`Expr`”的最大值。

> [!TIP] 
> 如果 `buildschema(json_column)` 指示语法错误：
>
> > *你的 `json_column` 是字符串，不是动态对象？*
>
> 则使用 `buildschema(parsejson(json_column))`。

## <a name="example"></a>示例

假定输入列具有三个动态值。

||
|---|
|`{"x":1, "y":3.5}`|
|`{"x":"somevalue", "z":[1, 2, 3]}`|
|`{"y":{"w":"zzz"}, "t":["aa", "bb"], "z":["foo"]}`|
||

生成的架构可能为：

```kusto
{ 
    "x":["int", "string"],
    "y":["double", {"w": "string"}],
    "z":{"`indexer`": ["int", "string"]},
    "t":{"`indexer`": "string"}
}
```

此架构表示：

* 根对象是具有四个属性（名为 x、y、z 和 t）的容器。
* “x”属性可能属于“int”类型或“string”类型。
* “y”属性可能属于“double”类型，或者另一个容器的“w”属性属于“string”类型。
* ``indexer`` 关键字指示“z”和“t”是数组。
* 数组“z”中的每一项均为“int”类型或“string”类型。
* “t”是一个字符串数组。
* 每个属性均为隐式可选，并且任何数组均可能为空。

### <a name="schema-model"></a>架构模型

返回架构的语法是：

```output
Container ::= '{' Named-type* '}';
Named-type ::= (name | '"`indexer`"') ':' Type;
Type ::= Primitive-type | Union-type | Container;
Union-type ::= '[' Type* ']';
Primitive-type ::= "int" | "string" | ...;
```

这些值等效于 TypeScript 类型批注的子集，作为 Kusto 动态值编码。 在 Typescript 中，示例架构应为：

```typescript
var someobject: 
{
    x?: (number | string),
    y?: (number | { w?: string}),
    z?: { [n:number] : (int | string)},
    t?: { [n:number]: string }
}
```
