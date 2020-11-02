---
title: 字符串运算符 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的字符串运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9755fd6d44bb80c1bee20b67e1f9d59254f05fc9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104426"
---
# <a name="string-operators"></a>字符串运算符

Kusto 提供了用于搜索字符串数据类型的各种查询运算符。 以下文章介绍如何为字符串词语编制索引、列出字符串查询运算符，以及提供用于优化性能的提示。

## <a name="understanding-string-terms"></a>了解字符串词语

Kusto 将为所有列（包括 `string` 类型的列）编制索引。 将根据实际数据为这些列构建多个索引。 这些索引不会直接公开，而是在查询中使用，此类查询的 `string` 运算符在其名称中包含 `has`（如 `has`、`!has`、`hasprefix`、`!hasprefix`）。 这些运算符的语义由列的编码方式决定。 这些运算符对 *词语* 进行匹配，而不是执行“纯”子字符串匹配。

### <a name="what-is-a-term"></a>什么是词语？ 

默认情况下，每个 `string` 值都分解为 ASCII 字母数字字符的最大序列，并将这些序列中的每一个都转换为一个词语。
例如，在下面的 `string` 中，词语是 `Kusto`、`WilliamGates3rd` 以及以下子字符串：`ad67d136`、`c1db`、`4f9f`、`88ef`、`d94f3b6b0b5a`。

```
Kusto:  ad67d136-c1db-4f9f-88ef-d94f3b6b0b5a;;WilliamGates3rd
```

Kusto 会构建一个词语索引，其中包含具有四个或更多个字符的所有词语。此索引由 `has`、`!has` 等使用。 如果查询查找小于四个字符的词语，或者使用 `contains` 运算符，则如果 Kusto 无法确定匹配项，它将恢复为扫描列中的值。 此方法比在词语索引中查找词语的速度要慢得多。

## <a name="operators-on-strings"></a>针对字符串的运算符

> [!NOTE]
> 下表中使用了以下缩写：
> * RHS = 表达式的右侧
> * LHS = 表达式的左侧

运算符        |描述                                                       |区分大小写|示例（生成 `true`）
----------------|------------------------------------------------------------------|--------------|-----------------------
`==`            |等于                                                            |是           |`"aBc" == "aBc"`
`!=`            |不等于                                                        |是           |`"abc" != "ABC"`
`=~`            |等于                                                            |否            |`"abc" =~ "ABC"`
`!~`            |不等于                                                        |否            |`"aBc" !~ "xyz"`
`has`           |右侧 (RHS) 是左侧 (LHS) 的整体     |否            |`"North America" has "america"`
`!has`          |RHS 不是 LHS 中的完整词语                                     |否            |`"North America" !has "amer"` 
`has_cs`        |RHS 是 LHS 中的完整词语                                        |是           |`"North America" has_cs "America"`
`!has_cs`       |RHS 不是 LHS 中的完整词语                                     |是           |`"North America" !has_cs "amer"` 
`hasprefix`     |RHS 是 LHS 中的词语前缀                                       |否            |`"North America" hasprefix "ame"`
`!hasprefix`    |RHS 不是 LHS 中的词语前缀                                   |否            |`"North America" !hasprefix "mer"` 
`hasprefix_cs`  |RHS 是 LHS 中的词语前缀                                       |是           |`"North America" hasprefix_cs "Ame"`
`!hasprefix_cs` |RHS 不是 LHS 中的词语前缀                                   |是           |`"North America" !hasprefix_cs "CA"` 
`hassuffix`     |RHS 是 LHS 中的词语后缀                                       |否            |`"North America" hassuffix "ica"`
`!hassuffix`    |RHS 不是 LHS 中的词语后缀                                   |否            |`"North America" !hassuffix "americ"`
`hassuffix_cs`  |RHS 是 LHS 中的词语后缀                                       |是           |`"North America" hassuffix_cs "ica"`
`!hassuffix_cs` |RHS 不是 LHS 中的词语后缀                                   |是           |`"North America" !hassuffix_cs "icA"`
`contains`      |RHS 以 LHS 子序列的形式存在                                |否            |`"FabriKam" contains "BRik"`
`!contains`     |LHS 中未出现 RHS                                         |否            |`"Fabrikam" !contains "xyz"`
`contains_cs`   |RHS 以 LHS 子序列的形式存在                                |是           |`"FabriKam" contains_cs "Kam"`
`!contains_cs`  |LHS 中未出现 RHS                                         |是           |`"Fabrikam" !contains_cs "Kam"`
`startswith`    |RHS 是 LHS 的初始子序列                              |否            |`"Fabrikam" startswith "fab"`
`!startswith`   |RHS 不是 LHS 的初始子序列                          |否            |`"Fabrikam" !startswith "kam"`
`startswith_cs` |RHS 是 LHS 的初始子序列                              |是           |`"Fabrikam" startswith_cs "Fab"`
`!startswith_cs`|RHS 不是 LHS 的初始子序列                          |是           |`"Fabrikam" !startswith_cs "fab"`
`endswith`      |RHS 是 LHS 的闭合子序列                               |否            |`"Fabrikam" endswith "Kam"`
`!endswith`     |RHS 不是 LHS 的闭合子序列                           |否            |`"Fabrikam" !endswith "brik"`
`endswith_cs`   |RHS 是 LHS 的闭合子序列                               |是           |`"Fabrikam" endswith_cs "kam"`
`!endswith_cs`  |RHS 不是 LHS 的闭合子序列                           |是           |`"Fabrikam" !endswith_cs "brik"`
`matches regex` |LHS 包含 RHS 的匹配项                                      |是           |`"Fabrikam" matches regex "b.*k"`
`in`            |等于某个元素                                     |是           |`"abc" in ("123", "345", "abc")`
`!in`           |不等于任何元素                                 |是           |`"bca" !in ("123", "345", "abc")`
`in~`           |等于某个元素                                     |否            |`"abc" in~ ("123", "345", "ABC")`
`!in~`          |不等于任何元素                                 |否            |`"bca" !in~ ("123", "345", "ABC")`
`has_any`       |与 `has` 相同，但适用于任何元素                    |否            |`"North America" has_any("south", "north")`

> [!TIP]
> 包含 `has` 的所有运算符都对四个或更多个字符的索引项进行搜索，而不对子字符串匹配项进行搜索。 通过将字符串分解为 ASCII 字母数字字符序列来创建词语。 请参阅[了解字符串词语](#understanding-string-terms)。

## <a name="performance-tips"></a>性能提示

为了获得更好的性能，当存在两个执行相同任务的运算符时，请使用区分大小写的那个运算符。
例如：

* 不要使用 `=~`，应使用 `==`
* 不要使用 `in~`，应使用 `in`
* 不要使用 `contains`，应使用 `contains_cs`

为了更快地得到结果，如果要测试是否存在符号或字母数字式字词（受非字母数字字符或字段的开头或结尾限制），请使用 `has` 或 `in`。 
`has` 执行起来比 `contains`、`startswith` 或 `endswith` 更快。

例如，下面的第一个查询运行速度更快：

```kusto
EventLog | where continent has "North" | count;
EventLog | where continent contains "nor" | count
```
