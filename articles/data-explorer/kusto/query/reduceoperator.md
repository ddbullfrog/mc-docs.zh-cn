---
title: reduce 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 reduce 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: ee9968c96324995b336e564115e757b5307c81e7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105215"
---
# <a name="reduce-operator"></a>reduce 运算符

基于值相似性组合一组字符串。

```kusto
T | reduce by LogMessage with threshold=0.1
```

对于每一个这样的组，它输出一个最能准确描述该组的模式（可能使用星号字符 `*` 表示通配符）、该组中值的计数以及该组的代表（该组中的某一原始值）  。

## <a name="syntax"></a>语法

*T* `|` `reduce` [`kind` `=` *ReduceKind* ] `by` *Expr* [`with` [`threshold` `=` *Threshold* ] [`,` `characters` `=` *Characters* ] ]

## <a name="arguments"></a>参数

* Expr：一个计算结果为 `string` 值的表达式。
* *阈值* ：范围 (0..1) 中的 `real` 文本。 默认为 0.1。 对于大型输入，阈值应较小。 
* *字符* ：`string` 文本，其中包含一系列字符，这些字符将添加到不会中断字词的字符列表中。 （例如，如果你希望 `aaa=bbbb` 和 `aaa:bbb` 各自为一个整体字词，而不是在 `=` 和 `:` 处中断，请使用 `":="` 作为字符串文本。）
* *ReduceKind* ：指定 reduce 风格。 目前唯一的有效值为 `source`。

## <a name="returns"></a>返回

该运算符返回一个表，其中包含三列（`Pattern`、`Count` 和 `Representative`），以及与组数相等的行。 `Pattern` 是组的模式值，`*` 用作通配符（表示任意插入字符串），`Count` 计算该模式所表示的运算符输入中的行的数量，而 `Representative` 是该组的输入中的一个值。

如果指定 `[kind=source]`，则运算符会将 `Pattern` 列追加到现有表结构。
请注意，该风格的语法架构可能会受后续更改的影响。

例如，`reduce by city` 的结果可能包括： 

|模式     |计数 |Representative|
|------------|------|--------------|
| San *      | 5182 |San Bernard   |
| Saint *    | 2846 |Saint Lucy    |
| Moscow     | 3726 |Moscow        |
| \* -on- \* | 2730 |One -on- One  |
| Paris      | 2716 |Paris         |

具有自定义词汇切分的另一个示例：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 1000 step 1
| project MyText = strcat("MachineLearningX", tostring(toint(rand(10))))
| reduce by MyText  with threshold=0.001 , characters = "X" 
```

|模式         |计数|Representative   |
|----------------|-----|-----------------|
|MachineLearning*|1000 |MachineLearningX4|

## <a name="examples"></a>示例

以下示例显示如何将 `reduce` 运算符应用于“sanitized”输入，在该输入中，要减少的列中 GUID 会在减少之前被替换

```kusto
// Start with a few records from the Trace table.
Trace | take 10000
// We will reduce the Text column which includes random GUIDs.
// As random GUIDs interfere with the reduce operation, replace them all
// by the string "GUID".
| extend Text=replace(@"[[:xdigit:]]{8}-[[:xdigit:]]{4}-[[:xdigit:]]{4}-[[:xdigit:]]{4}-[[:xdigit:]]{12}", @"GUID", Text)
// Now perform the reduce. In case there are other "quasi-random" identifiers with embedded '-'
// or '_' characters in them, treat these as non-term-breakers.
| reduce by Text with characters="-_"
```

## <a name="see-also"></a>请参阅

[autocluster](./autoclusterplugin.md)

**备注**

`reduce` 运算符的实现很大程度上基于 Risto Vaarandi 所著论文[用于从事件日志中挖掘模式的数据聚类分析算法](https://ristov.github.io/publications/slct-ipom03-web.pdf)。
