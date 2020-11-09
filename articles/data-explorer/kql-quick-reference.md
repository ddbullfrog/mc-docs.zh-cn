---
title: KQL 快速参考
description: 有用的 KQL 函数的列表及其定义与语法示例。
author: orspod
ms.author: v-tawe
ms.reviewer: ''
ms.service: data-explorer
ms.topic: conceptual
origin.date: 01/19/2020
ms.date: 09/30/2020
ms.openlocfilehash: 5ce52ca6abd91f374e283c085a051282651546ac
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105068"
---
# <a name="kql-quick-reference"></a>KQL 快速参考

本文显示了函数的列表及其说明，以帮助你开始使用 Kusto 查询语言。

| 运算符/函数                               | 说明                           | 语法                                           |
| :---------------------------------------------- | :------------------------------------ |:-------------------------------------------------|
|**筛选/搜索/条件**                      |**_通过筛选或搜索来查找相关数据_** |                      |
| [where](kusto/query/whereoperator.md)                      | 基于特定的谓词进行筛选           | `T | where Predicate`                         |
| [where contains/has](kusto/query/whereoperator.md)        | `Contains`：查找任何子字符串匹配项 <br> `Has`：查找特定字词（性能更好）  | `T | where col1 contains/has "[search term]"`|
| [search](kusto/query/searchoperator.md)                    | 在表的所有列中搜索值 | `[TabularSource |] search [kind=CaseSensitivity] [in (TableSources)] SearchPredicate` |
| [take](kusto/query/takeoperator.md)                        | 返回指定数量的记录。 用来测试查询<br>**_注意_** ：`_take`_ 和 `_limit`_ 是同义词。 | `T | take NumberOfRows` |
| [case](kusto/query/casefunction.md)                        | 添加一个条件语句，类似于其他系统中的 if/then/elseif。 | `case(predicate_1, then_1, predicate_2, then_2, predicate_3, then_3, else)` |
| [distinct](kusto/query/distinctoperator.md)                | 生成一个表，其中包含输入表中所提供列的不同组合 | `distinct [ColumnName], [ColumnName]` |
| **日期/时间**                                   |**_使用日期和时间函数的操作_**               |                          |
|[ago](kusto/query/agofunction.md)                           | 返回相对于查询执行时间的时间偏移量。 例如，`ago(1h)` 是当前时钟读数之前的一小时。 | `ago(a_timespan)` |
| [format_datetime](kusto/query/format-datetimefunction.md)  | 以[各种日期格式](kusto/query/format-datetimefunction.md#supported-formats)返回数据。 | `format_datetime(datetime , format)` |
| [bin](kusto/query/binfunction.md)                          | 将某个时间范围内的所有值进行舍入并对其进行分组 | `bin(value,roundTo)` |
| **创建/删除列**                   |**_在表中添加或删除列_** |                                                    |
| [print](kusto/query/printoperator.md)                      | 输出包含一个或多个标量表达式的单个行 | `print [ColumnName =] ScalarExpression [',' ...]` |
| [project](kusto/query/projectoperator.md)                  | 选择要按指定顺序包括的列 | `T | project ColumnName [= Expression] [, ...]` <br> 或 <br> `T | project [ColumnName | (ColumnName[,]) =] Expression [, ...]` |
| [project-away](kusto/query/projectawayoperator.md)         | 选择要从输出中排除的列 | `T | project-away ColumnNameOrPattern [, ...]` |
| [project-keep](kusto/query/project-keep-operator.md)         | 选择要在输出中保留的列 | `T | project-keep ColumnNameOrPattern [, ...]` |
| [project-rename](kusto/query/projectrenameoperator.md)     | 对结果输出中的列重命名 | `T | project-rename new_column_name = column_name` |
| [project-reorder](kusto/query/projectreorderoperator.md)   | 对结果输出中的列重新排序 | `T | project-reorder Col2, Col1, Col* asc` |
| [extend](kusto/query/extendoperator.md)                    | 创建一个计算列并将其添加到结果集 | `T | extend [ColumnName | (ColumnName[, ...]) =] Expression [, ...]` |
| **对数据集进行排序和聚合**                 |**_通过以有意义的方式对数据进行排序或分组来重构数据_**|                  |
| [sort](kusto/query/sortoperator.md)                        | 根据一个或多个列按升序或降序为输入表的行排序 | `T | sort by expression1 [asc|desc], expression2 [asc|desc], …` |
| [返回页首](kusto/query/topoperator.md)                          | 当使用 `by` 对数据集进行排序时返回数据集的前 N 行 | `T | top numberOfRows by expression [asc|desc] [nulls first|last]` |
| [summarize](kusto/query/summarizeoperator.md)              | 根据 `by` 分组列对行进行分组，并计算每个组的聚合 | `T | summarize [[Column =] Aggregation [, ...]] [by [Column =] GroupExpression [, ...]]` |
| [count](kusto/query/countoperator.md)                       | 对输入表中的记录进行计数（例如 T）<br>此运算符是 `summarize count() ` 的简写| `T | count` |
| [join](kusto/query/joinoperator.md)                        | 通过匹配每个表中指定列的值，合并两个表的行以组成新表。 支持完整范围的联接类型：`flouter`、`inner`、`innerunique`、`leftanti`、`leftantisemi`、`leftouter`、`leftsemi`、`rightanti`、`rightantisemi`、`rightouter`、`rightsemi` | `LeftTable | join [JoinParameters] ( RightTable ) on Attributes` |
| [union](kusto/query/unionoperator.md)                      | 获取两个或多个表，并返回表中的所有行。 | `[T1] | union [T2], [T3], …` |
| [range](kusto/query/rangeoperator.md)                      | 生成包含一系列算术值的表 | `range columnName from start to stop step step` |
| **设置数据格式**                                 | **_重构数据以便以有用的方式输出_** | |
| [lookup](kusto/query/lookupoperator.md)                    | 使用在维度表中查找的值扩展事实数据表的列 | `T1 | lookup [kind = (leftouter|inner)] ( T2 ) on Attributes` |
| [mv-expand](kusto/query/mvexpandoperator.md)               | 将动态数组转换为行（多值扩展） | `T | mv-expand Column` |
| [parse](kusto/query/parseoperator.md)                      | 计算字符串表达式并将其值分析为一个或多个计算列。 用于构造非结构化数据。 | `T | parse [kind=regex  [flags=regex_flags] |simple|relaxed] Expression with * (StringConstant ColumnName [: ColumnType]) *...` |
| [make-series](kusto/query/make-seriesoperator.md)          | 沿指定的轴创建指定聚合值的系列 | `T | make-series [MakeSeriesParamters] [Column =] Aggregation [default = DefaultValue] [, ...] on AxisColumn from start to end step step [by [Column =] GroupExpression [, ...]]` |
| [let](kusto/query/letstatement.md)                         | 将名称绑定到可引用其绑定值的表达式。 值可以是 lambda 表达式，用来创建作为查询的一部分的即席函数。 使用 `let` 基于其结果看起来像新表的表创建表达式。 | `let Name = ScalarExpression | TabularExpression | FunctionDefinitionExpression` |
| **常规**                                     | **_其他操作和函数_** | |
| [invoke](kusto/query/invokeoperator.md)                    | 对作为输入的表运行此函数。 | `T | invoke function([param1, param2])` |
| [evaluate pluginName](kusto/query/evaluateoperator.md)     | 评估查询语言扩展（插件） | `[T |] evaluate [ evaluateParameters ] PluginName ( [PluginArg1 [, PluginArg2]... )` |
| **可视化**                               | **_以图形格式显示数据的操作_** | |
| [render](kusto/query/renderoperator.md) | 将结果呈现为图形输出 | `T | render Visualization [with (PropertyName = PropertyValue [, ...] )]` |
