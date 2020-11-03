---
title: pattern 语句 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 pattern 语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: d8d89516a01adf8b321577c63893fe4d75ddd059
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106306"
---
# <a name="pattern-statement"></a>pattern 语句

::: zone pivot="azuredataexplorer"

**模式** 是一种命名视图样式的构造，用于将预定义的字符串元组映射到无参数函数正文。 模式在两个方面是独一无二的：

* “调用”模式时使用的语法类似于具有作用域的表引用。
* 模式具有一组可以映射的受控封闭式参数值，映射过程由 Kusto 完成。 如果声明了某个模式但未定义它，则 Kusto 会识别对该模式的所有调用并将其标记为错误。 进行此识别后，中间层应用程序可以“解析”这些模式。

## <a name="pattern-declaration"></a>模式声明

pattern 语句用来声明或定义模式。
例如，某个 pattern 语句将 `app` 声明为模式。

```kusto
declare pattern app;
```

此语句告诉 Kusto `app` 是一种模式，但并未告诉 Kusto 如何解析该模式。 因此，在查询中调用此模式的任何尝试都会导致特定的错误，并会列出所有此类调用。 例如：

```kusto
declare pattern app;
app("ApplicationX").StartEvents
| join kind=inner app("ApplicationX").StopEvents on CorrelationId
| count
```

此查询会在 Kusto 中生成错误，指示无法解析接下来的模式调用：`app("ApplicationX")["StartEvents"]` 和 `app("ApplicationX")["StopEvents"]`。

## <a name="syntax"></a>语法

`declare` `pattern` *PatternName*

## <a name="pattern-definition"></a>模式定义

pattern 语句还可以用来定义模式。 在模式定义中，所有可能的模式调用都被显式列出，并给出相应的表格表达式。 然后，当 Kusto 执行查询时，它会将每个模式调用替换为对应的模式正文。 例如：

```kusto
declare pattern app = (applicationId:string)[eventType:string]
{
    ("ApplicationX").["StopEvents"] = { database("AppX").Events | where EventType == "StopEvent" };
    ("ApplicationX").["StartEvents"] = { database(applicationId).Events | where EventType == eventType } ;
};
app("ApplicationX").StartEvents
| join kind=inner app("ApplicationX").StopEvents on CorrelationId
| count
```

为每个匹配的模式提供的表达式是表名或对 [let 语句](letstatement.md)的引用。

## <a name="syntax"></a>语法

`declare` `pattern` *PatternName* = `(`*ArgName* `:` *ArgType* [`,` ... ]`)` [`[` *PathName* `:` *PathArgType* `]`] `{`
&nbsp;&nbsp;&nbsp;&nbsp; `(` *ArgValue1* [`,` *ArgValue2* ... ] `)` [ `.[` *PathValue `]` ] `=` `{`  *expression*  `};` &nbsp;&nbsp;&nbsp;&nbsp; [ &nbsp;&nbsp;&nbsp;&nbsp; `(` *ArgValue1_2* [`,` *ArgValue2_2* ... ] `)` [ `.[` *PathValue_2* `]` ] `=` `{`  *expression_2*  `};` &nbsp;&nbsp;&nbsp;&nbsp; ... &nbsp;&nbsp;&nbsp;&nbsp; ] `}`

* PatternName：模式关键字的名称。 仅允许使用定义关键字的语法：用于检测具有指定关键字的所有模式引用。
* ArgName：模式参数的名称。 模式允许一个或多个参数名称。
* ArgType：模式参数的类型（当前仅允许 `string`）
* PathName：路径参数的名称。 模式允许零个或一个路径名称。
* PathType：路径参数的类型（当前仅允许 `string`）
* ArgValue1, ArgValue2, ... - 模式参数的值（当前仅允许 `string` 文本）
* PathValue - 模式路径的值（当前仅允许 `string` 文本）
* expression：expression 为表格表达式（例如 `Logs | where Timestamp > ago(1h)`），或者为引用某个函数的 lambda 表达式。

## <a name="pattern-invocation"></a>模式调用

模式调用语法类似于具有作用域的表引用语法。

* *PatternName* `(` *ArgValue1* [`,` *ArgValue2* ...] `).`*PathValue*
* *PatternName* `(` *ArgValue1* [`,` *ArgValue2* ...] `).["`*PathValue*`"]`

## <a name="notes"></a>说明

**方案**

pattern 语句是针对中间层应用程序设计的，这类应用程序接受用户查询然后将这些查询发送到 Kusto。 这类应用程序通常以逻辑架构模型作为那些用户查询的前缀。 模型是一组 [let 语句](letstatement.md)，可能以某个 [restrict 语句](restrictstatement.md)作为后缀。

某些应用程序需要可以为用户提供的语法。 语法用来引用在应用程序构造的逻辑架构中定义的实体。 但是，有时不会提前知道实体，或者潜在的实体数量太大，无法在逻辑架构中预定义。

模式通过以下方式解决了这种情况。 中间层应用程序将查询发送到 Kusto，并声明所有模式，但不定义。 然后，Kusto 分析该查询。 如果有一个或多个模式调用，则 Kusto 会向中间层应用程序返回一个错误，其中会显式列出所有此类调用。 然后，中间层应用程序可以解析每个引用，然后重新运行查询。 这次将添加完整的细化模式定义作为前缀。

**规范化**

Kusto 会自动将模式规范化。 例如，下面是对同一模式的所有调用，并报告了单个模式。

```kusto
declare pattern app;
union
  app("ApplicationX").StartEvent,
  app('ApplicationX').StartEvent,
  app("ApplicationX").['StartEvent'],
  app("ApplicationX").["StartEvent"]
```

这也意味着，不能将它们一起定义，因为它们被视为是同一个。

**通配符**

Kusto 不会以任何特殊方式对模式中的通配符进行处理。 例如，在以下查询中：

```kusto
declare pattern app;
union app("ApplicationX").*
| count
```

Kusto 会报告单个缺少的模式调用：`app("ApplicationX").["*"]`。

## <a name="examples"></a>示例

基于多个模式调用进行查询。

```kusto
declare pattern A
{
    // ...
};
union (A('a1').Text), (A('a2').Text)
```

|应用|SomeText|
|---|---|
|应用 #1|这是一个自由文本：1|
|应用 #1|这是一个自由文本：2|
|应用 #1|这是一个自由文本：3|
|应用 #1|这是一个自由文本：4|
|应用 #1|这是一个自由文本：5|
|应用 #2|这是一个自由文本：9|
|应用 #2|这是一个自由文本：8|
|应用 #2|这是一个自由文本：7|
|应用 #2|这是一个自由文本：6|
|应用 #2|这是一个自由文本：5|

```kusto
declare pattern App;
union (App('a1').Text), (App('a2').Text)
```

**语义错误** ：

> SEM0036:未声明一个或多个模式引用。 检测到的模式引用: ["App('a1').['Text']","App('a2').['Text']"]。

```kusto
declare pattern App;
declare pattern App = (applicationId:string)[scope:string]  
{
    ('a1').['Data']    = { range x from 1 to 5 step 1 | project App = "App #1", Data    = x };
    ('a1').['Metrics'] = { range x from 1 to 5 step 1 | project App = "App #1", Metrics = rand() };
    ('a2').['Data']    = { range x from 1 to 5 step 1 | project App = "App #2", Data    = 10 - x };
    ('a3').['Metrics'] = { range x from 1 to 5 step 1 | project App = "App #3", Metrics = rand() };
};
union (App('a2').Metrics), (App('a3').Metrics) 
```

**返回了语义错误** ：

> SEM0036:未声明一个或多个模式引用。 检测到的模式引用: ["App('a2').['Metrics']","App('a3').['Metrics']"]。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
