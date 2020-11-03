---
title: search 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 search 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: f128828a76fdcd7d9226185b2abf8ac44c8422ae
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106192"
---
# <a name="search-operator"></a>search 运算符

search 运算符提供了多表/多列搜索体验。

## <a name="syntax"></a>语法

* [ *TabularSource* `|`] `search` [`kind=`*CaseSensitivity* ] [`in` `(`*TableSources*`)`] *SearchPredicate*

## <a name="arguments"></a>参数

* TabularSource：一个可选的表格表达式，充当要在其中进行搜索的数据源，例如表名称、[union 运算符](unionoperator.md)、表格查询结果，等等。不能与包括 TableSources 的可选短语一起出现。

* CaseSensitivity：一个可选的标志，它控制所有 `string` 标量运算符在区分大小写方面的行为。 有效值为两个同义词 `default` 和 `case_insensitive`（后者是 `has` 之类的运算符的默认值，也就是不区分大小写）以及 `case_sensitive`（这将强制所有此类运算符转换为区分大小写的匹配模式）。

* TableSources：要参与搜索的“带通配符的”表名的逗号分隔列表（可选）。
  此列表具有与 [union 运算符](unionoperator.md)列表相同的语法。
  不能与可选的 TabularSource一起出现。

* SearchPredicate：一个必需的谓词，用于定义要搜索的内容（换言之，这是要针对输入中的每条记录进行评估的布尔表达式，如果它返回 `true`，则会输出该记录。）SearchPredicate 的语法扩展并修改布尔表达式的常规 Kusto 语法：

  **字符串匹配扩展** ：作为 SearchPredicate 中的词语出现的字符串文本表示在所有列与使用 `has`、`hasprefix`、`hassuffix` 及这些运算符的反向 (`!`) 或区分大小写 (`sc`) 版本的文本之间存在词语匹配。 是应用 `has`、`hasprefix` 还是 `hassuffix` 取决于文本是否以星号 (`*`) 开头和/或结尾。 不允许在文本内部使用星号。

    |文本   |运算符   |
    |----------|-----------|
    |`billg`   |`has`      |
    |`*billg`  |`hassuffix`|
    |`billg*`  |`hasprefix`|
    |`*billg*` |`contains` |
    |`bi*lg`   |`matches regex`|

  **列限制** ：默认情况下，字符串匹配扩展会尝试匹配数据集的所有列。 可以使用以下语法将此匹配限制为特定列： *ColumnName*`:`*StringLiteral* 。

  **字符串等式** ：可以使用 *ColumnName*`==`*StringLiteral* 语法根据字符串值对列进行精确匹配（而不是字词匹配）。

  **其他布尔表达式** ：此语法支持所有正则 Kusto 布尔表达式。
    例如，`"error" and x==123` 表示：搜索在任何列中具有词语 `error` 并且在 `x` 列中包含值 `123` 的记录。

  **正则表达式匹配** ：正则表达式匹配是使用 Column `matches regex` StringLiteral 语法进行指示的，其中 StringLiteral 是正则表达式模式。

请注意，如果同时省略 TabularSource 和 TableSources，则会对作用域中数据库的所有不受限制的表和视图执行搜索。

## <a name="summary-of-string-matching-extensions"></a>字符串匹配扩展汇总

  |# |语法                                 |含义（等效的 `where`）           |注释|
  |--|---------------------------------------|---------------------------------------|--------|
  | 1|`search "err"`                         |`where * has "err"`                    ||
  | 2|`search in (T1,T2,A*) and "err"`       |<code>union T1,T2,A* &#124; where * has "err"<code>   ||
  | 3|`search col:"err"`                     |`where col has "err"`                  ||
  | 4|`search col=="err"`                    |`where col=="err"`                     ||
  | 5|`search "err*"`                        |`where * hasprefix "err"`              ||
  | 6|`search "*err"`                        |`where * hassuffix "err"`              ||
  | 7|`search "*err*"`                       |`where * contains "err"`               ||
  | 8|`search "Lab*PC"`                      |`where * matches regex @"\bLab.*PC\b"`||
  | 9|`search *`                             |`where 0==0`                           ||
  |10|`search col matches regex "..."`       |`where col matches regex "..."`        ||
  |11|`search kind=case_sensitive`           |                                       |所有字符串比较都区分大小写|
  |12|`search "abc" and ("def" or "hij")`    |`where * has "abc" and (* has "def" or * has hij")`||
  |13|`search "err" or (A>a and A<b)`        |`where * has "err" or (A>a and A<b)`   ||

## <a name="remarks"></a>备注

**不同于** [find 运算符](findoperator.md)，`search` 运算符不支持以下项：

1. `withsource=`：输出将始终包含一个名为 `$table` 且类型为 `string` 的列，其值是要从中检索每个记录的表名称（或者，如果源不是一个表，而是一个复合表达式，则其值为系统生成的某个名称）。
2. `project=`、`project-smart`：此输出架构等效于 `project-smart` 输出架构。

## <a name="examples"></a>示例

```kusto
// 1. Simple term search over all unrestricted tables and views of the database in scope
search "billg"

// 2. Like (1), but looking only for records that match both terms
search "billg" and ("steveb" or "satyan")

// 3. Like (1), but looking only in the TraceEvent table
search in (TraceEvent) and "billg"

// 4. Like (2), but performing a case-sensitive match of all terms
search "BillB" and ("SteveB" or "SatyaN")

// 5. Like (1), but restricting the match to some columns
search CEO:"billg" or CSA:"billg"

// 6. Like (1), but only for some specific time limit
search "billg" and Timestamp >= datetime(1981-01-01)

// 7. Searches over all the higher-ups
search in (C*, TF) "billg" or "davec" or "steveb"

// 8. A different way to say (7). Prefer to use (7) when possible
union C*, TF | search "billg" or "davec" or "steveb"
```

## <a name="performance-tips"></a>性能提示

  |# |提示                                                                                  |Prefer                                        |Over                                                                    |
  |--|-------------------------------------------------------------------------------------|----------------------------------------------|------------------------------------------------------------------------|
  | 1| 最好使用单个 `search` 运算符而非多个连续的 `search` 运算符|`search "billg" and ("steveb" or "satyan")`   |<code>search "billg" &#124; search "steveb" or "satyan"<code>           ||
  | 2| 最好在 `search` 运算符内进行筛选                                       |`search "billg" and "steveb"`                 |<code>search * &#124; where * has "billg" and * has "steveb"<code>      ||
