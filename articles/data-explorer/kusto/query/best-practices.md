---
title: 查询最佳做法 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的查询最佳做法。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/03/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9cc2dc213f0827739cd39738d684ef57ea1eb02e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104037"
---
# <a name="query-best-practices"></a>查询最佳做法

下面是提高查询运行速度的几个最佳做法。

|操作  |用途  |不要使用  |注释  |
|---------|---------|---------|---------|
| **时间筛选器** | 首先使用时间筛选器。 ||Kusto 经过高度优化，可以使用时间筛选器。| 
|**字符串运算符**      | 使用 `has` 运算符     | 不要使用 `contains`     | 查找完整标记时，`has` 效果更好，因为它不会查找子字符串。   |
|**区分大小写的运算符**     |  使用 `==`       | 不要使用 `=~`       |  如果可能，请使用区分大小写的运算符。       |
| | 使用 `in` | 不要使用 `in~`|
|  | 使用 `contains_cs`         | 不要使用 `contains`        | 如果可以使用 `has`/`has_cs` 而不使用 `contains`/`contains_cs`，则更好。 |
| **搜索文本**    |    查找特定列     |    不要使用 `*`    |   `*` 对所有列执行全文搜索。    |
| **从数百万行的 [动态对象](./scalar-data-types/dynamic.md)中提取字段**    |  如果大多数查询从数百万行的动态对象中提取字段，则会在引入时具体化列。      |         | 这样，只需为列提取付费一次。    |
| **具有多次使用的值的 `let` 语句** | 使用 [materialize() 函数](./materializefunction.md) |  |   有关如何使用 `materialize()` 的详细信息，请参阅 [materialize()](materializefunction.md)。|
| **对超过 10 亿条记录应用转换**| 调整查询以减少馈送到转换中的数据量。| 如果可以避免，请勿转换大量数据。 | |
| **新查询** | 在末尾使用 `limit [small number]` 或 `count`。 | |     对未知数据集运行未绑定的查询，可能会产生要返回到客户端的 GB 级结果，从而导致响应缓慢和群集忙碌。|
| **不区分大小写的比较** | 使用 `Col =~ "lowercasestring"` | 不要使用 `tolower(Col) == "lowercasestring"` |
| **比较已小写（或大写）的数据** | `Col == "lowercasestring"`（或 `Col == "UPPERCASESTRING"`） | 避免使用不区分大小写的比较。||
| **按列筛选** |  按表列筛选。|不要按计算列进行筛选。 | |
| | 使用 `T | where predicate(<expression>)` | 不要使用 `T | extend _value = <expression> | where predicate(_value)` ||
| **summarize 运算符** |  当 summarize 运算符的 `group by keys` 具有高基数时，请使用 [hint.strategy=shuffle](./shufflequery.md)。 | | 理想情况下，高基数高于 100 万。|
|**[join 运算符](./joinoperator.md)** | 选择行数较少的表作为第一个（查询中最左侧）。 ||
| 跨群集联接 |在群集中，在联接的“右”侧（大多数数据位于此处）运行查询。 ||
|左侧较小，右侧较大时进行联接 | 使用 [hint.strategy=broadcast](./broadcastjoin.md) || 小是指最多 100,000 条记录。 |
|当两侧都过大时进行联接 | 使用 [hint.strategy=shuffle](./shufflequery.md) || 当联接键具有高基数时使用。|
|**提取具有相同格式或模式的字符串的列上的值**|  使用 [parse 运算符](./parseoperator.md) | 不要使用几个 `extract()` 语句。  | 例如，像 `"Time = <time>, ResourceId = <resourceId>, Duration = <duration>, ...."` 这样的值
|**[extract() 函数](./extractfunction.md)**| 当分析的字符串不都遵循相同的格式或模式时使用。| |使用 REGEX 提取所需的值。|
| **[materialize() 函数](./materializefunction.md)** | 推送所有可能的运算符，这些运算符将减少具体化的数据集，并且仍然保留查询的语义。 | |例如，筛选器或仅项目所需的列。

