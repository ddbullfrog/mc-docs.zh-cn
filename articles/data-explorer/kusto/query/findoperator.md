---
title: find 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 find 运算符。
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
ms.openlocfilehash: 1584702311952a0b48b276e44b3ad7fe26f3199d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105143"
---
# <a name="find-operator"></a>find 运算符

在一组表中查找与谓词匹配的行。

::: zone pivot="azuredataexplorer"

`find` 的作用域也可以跨数据库或跨群集。

```kusto
find in (Table1, Table2, Table3) where Fruit=="apple"

find in (database('*').*) where Fruit == "apple"

find in (cluster('cluster_name').database('MyDB*'.*)) where Fruit == "apple"
```

::: zone-end

::: zone pivot="azuremonitor"

```kusto
find in (Table1, Table2, Table3) where Fruit=="apple"
```

::: zone-end

## <a name="syntax"></a>语法

* `find` [`withsource`=ColumnName] [`in` `(`Table [`,` Table, ...]`)`] `where` Predicate [`project-smart` | `project` ColumnName [`:`ColumnType] [`,` ColumnName[`:`ColumnType], ...][`,` `pack(*)`]]        

* `find` Predicate [`project-smart` | `project` ColumnName[`:`ColumnType] [`,` ColumnName[`:`ColumnType], ...] [`, pack(*)`]]     

## <a name="arguments"></a>参数

::: zone pivot="azuredataexplorer"

* `withsource=`ColumnName：可选。 默认情况下，输出将包括一个名为 source_ 的列，其值指示哪个源表提供了每一行。 如果指定，将使用 ColumnName，而不是 source_ 。
通配符匹配后，如果查询引用了多个数据库（包括默认数据库）中的表，则此列的值将具有使用数据库进行限定的表名。 同样，如果引用了多个群集，则值中将存在“群集”和“数据库”限定条件 。
* *谓词* ：针对输入表 Table [`,` Table, ...] 的列的 `boolean` [表达式](./scalar-data-types/bool.md) 。对每个输入表中的每一行执行计算。 有关详细信息，请参阅[谓词语法详细信息](./findoperator.md#predicate-syntax)。
* `Table`：可选。 默认情况下，find 将在当前数据库的所有表中查找：
    *  表的名称，如 `Events`
    *  查询表达式，如 `(Events | where id==42)`
    *  使用通配符指定的一组表。 例如，`E*` 将构成数据库中所有名称以 `E` 开头的表的并集。
* `project-smart` | `project`：如果未指定，则默认情况下将使用 `project-smart`。 有关详细信息，请参阅[输出架构详细信息](./findoperator.md#output-schema)。

::: zone-end

::: zone pivot="azuremonitor"

* `withsource=`ColumnName：可选。 默认情况下，输出将包括一个名为 source_ 的列，其值指示哪个源表提供了每一行。 如果指定，将使用 ColumnName，而不是 source_ 。
* *谓词* ：针对输入表 Table [`,` Table, ...] 的列的 `boolean` [表达式](./scalar-data-types/bool.md) 。对每个输入表中的每一行执行计算。 有关详细信息，请参阅[谓词语法详细信息](./findoperator.md#predicate-syntax)。
* `Table`：可选。 默认情况下 find 将在所有表中搜索：
    *  表的名称，如 `Events` 
    *  查询表达式，如 `(Events | where id==42)`
    *  使用通配符指定的一组表。 例如，`E*` 将构成所有名称以 `E` 开头的表的并集。
* `project-smart` | `project`：如果未指定，则默认情况下将使用 `project-smart`。 有关详细信息，请参阅[输出架构详细信息](./findoperator.md#output-schema)。

::: zone-end

## <a name="returns"></a>返回

Predicate 为 `true` 的 Table [`,` Table, ...] 中的行的转换  。 行根据[输出架构](#output-schema)进行转换。

## <a name="output-schema"></a>输出架构

**source_ 列**

find 运算符输出始终包含具有源表名称的 source_ 列。 可以使用 `withsource` 参数重命名该列。

**结果列**

将筛除不包含谓词评估所用的任何列的源表。

使用 `project-smart` 时，输出中会显示的列为：
* 在谓词中显式显示的列。
* 所有筛选的表的公共列。

其余列将打包为属性包，并显示在其他 `pack_` 列中。
谓词显式引用并显示在具有多种类型（每种类型在结果架构中将具有不同的列）的多个表中的列。 每个列名都基于原始列名和类型构造而成，并用下划线分隔。

使用 `project` ColumnName[`:`ColumnType] [`,` ColumnName[`:`ColumnType], ...][`,` `pack(*)`] 时   ：
* 结果表将包含列表中指定的列。 如果源表不包含特定列，则相应行中的值将为 NULL。
* 使用 ColumnName 指定 ColumnType 时，“结果”中的此列将具有给定的类型，并且值将根据需要强制转换为该类型 。 评估 Predicate 时，强制转换不会影响列类型。
* 使用 `pack(*)` 时，其余列将打包为属性包，并显示在其他 `pack_` 列中。

**pack_ 列**

此列将包含一个属性包，其中包含输出架构中未显示的所有列的数据。 源列名将用作属性名，列值将用作属性值。

## <a name="predicate-syntax"></a>谓词语法

find 运算符支持 `* has` 术语的替代语法，并且仅使用 term 即可在所有输入列中搜索术语 。

有关某些筛选函数的摘要，请参阅 [where 运算符](./whereoperator.md)。

## <a name="notes"></a>说明

* 如果 `project` 子句引用了出现在多个表中且具有多种类型的列，则在 project 子句中，类型必须遵循此列引用
* 如果某列出现在多个表中且具有多种类型，并且正在使用 `project-smart`，则 `find` 的结果中每种类型都会有一个对应的列，如 [union](./unionoperator.md) 中所述
* 当使用 project-smart 时，谓词、源表集或表架构中的更改可能会导致输出架构发生更改。 如果需要常量结果架构，请改用 project
* `find` 作用域不能包括[函数](../management/functions.md)。 要在 find 作用域中包含函数，请使用 [view 关键字](./letstatement.md)定义 [let 语句](./letstatement.md)。

## <a name="performance-tips"></a>性能提示

* 使用[表](../management/tables.md)，而不是[表格表达式](./tabularexpressionstatements.md)。
如果使用表格表达式，则 find 运算符将回退到 `union` 查询，这可能会导致性能下降。
* 如果列出现在多个表中并具有多种类型，并且是 project 子句的一部分，则在将表传递给 `find` 之前，最好将 ColumnType 添加到 project 子句，而不是修改表。
* 向谓词添加基于时间的筛选器。 使用日期/时间列值或 [ingestion_time()](./ingestiontimefunction.md)。
* 搜索特定列而不是全文搜索。
* 最好不要引用出现在多个表中且具有多个类型的列。 如果在为多个类型解析此类列类型时谓词有效，则查询将回退到并集。
例如，请参阅[将 find 用作 union 的案例示例](./findoperator.md#examples-of-cases-where-find-will-act-as-union)。
 
## <a name="examples"></a>示例

::: zone pivot="azuredataexplorer"

### <a name="term-lookup-across-all-tables-in-current-database"></a>在当前数据库的所有表中进行字词查找

以下查询从当前数据库的所有表中查找其中的任意列包含单词 `Kusto` 的所有行。
生成的记录根据[输出架构](#output-schema)进行转换。

```kusto
find "Kusto"
```

## <a name="term-lookup-across-all-tables-matching-a-name-pattern-in-the-current-database"></a>对当前数据库中与名称模式匹配的所有表进行字词查找

以下查询从当前数据库中名称以 `K` 开头的所有表中，查找其中的任意列包含单词 `Kusto` 的所有行。
生成的记录根据[输出架构](#output-schema)进行转换。

```kusto
find in (K*) where * has "Kusto"
```

### <a name="term-lookup-across-all-tables-in-all-databases-in-the-cluster"></a>对群集中所有数据库的所有表进行字词查找

以下查询从所有数据库的所有表中查找其中的任意列包含单词 `Kusto` 的所有行。
此查询是一个[跨数据库](./cross-cluster-or-database-queries.md)查询。
生成的记录根据[输出架构](#output-schema)进行转换。

```kusto
find in (database('*').*) "Kusto"
```

### <a name="term-lookup-across-all-tables-and-databases-matching-a-name-pattern-in-the-cluster"></a>对群集中与名称模式匹配的所有表和数据库进行字词查找

以下查询从所有名称以 `K` 开头的表中和所有名称以 `B` 开头的数据库中，查找其中的任意列包含单词 `Kusto` 的所有行。
生成的记录根据[输出架构](#output-schema)进行转换。

```kusto
find in (database("B*").K*) where * has "Kusto"
```

### <a name="term-lookup-in-several-clusters"></a>在多个群集中进行字词查找

以下查询从所有名称以 `K` 开头的表中和所有名称以 `B` 开头的数据库中，查找其中的任意列包含单词 `Kusto` 的所有行。
生成的记录根据[输出架构](#output-schema)进行转换。

```kusto
find in (cluster("cluster1").database("B*").K*, cluster("cluster2").database("C*".*))
where * has "Kusto"
```

::: zone-end

::: zone pivot="azuremonitor"

### <a name="term-lookup-across-all-tables"></a>跨所有表进行字词查找

以下查询从所有表中查找其中的任意列包含单词 `Kusto` 的所有行。
生成的记录根据[输出架构](#output-schema)进行转换。

```kusto
find "Kusto"
```

::: zone-end

## <a name="examples-of-find-output-results"></a>`find` 输出结果的示例  

下面的示例演示如何针对两个表使用 `find`：EventsTable1 和 EventsTable2 。
假设我们有这两个表的以下内容：

### <a name="eventstable1"></a>EventsTable1

|Session_Id|级别|EventText|版本
|---|---|---|---|
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|信息|Some Text1|v1.0.0
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|错误|Some Text2|v1.0.0
|28b8e46e-3c31-43cf-83cb-48921c3986fc|错误|Some Text3|v1.0.1
|8f057b11-3281-45c3-a856-05ebb18a3c59|信息|Some Text4|v1.1.0

### <a name="eventstable2"></a>EventsTable2

|Session_Id|级别|EventText|EventName
|---|---|---|---|
|f7d5f95f-f580-4ea6-830b-5776c8d64fdd|信息|Some Other Text1|Event1
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|信息|Some Other Text2|Event2
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|错误|Some Other Text3|Event3
|15eaeab5-8576-4b58-8fc6-478f75d8fee4|错误|Some Other Text4|Event4

### <a name="search-in-common-columns-project-common-and-uncommon-columns-and-pack-the-rest"></a>搜索公共列、投影通用列和非通用列，然后打包其余列  

```kusto
find in (EventsTable1, EventsTable2) 
     where Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e' and Level == 'Error' 
     project EventText, Version, EventName, pack(*)
```

|source_|EventText|版本|EventName|pack_
|---|---|---|---|---|
|EventsTable1|Some Text2|v1.0.0||{"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error"}
|EventsTable2|Some Other Text3||Event3|{"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error"}


### <a name="search-in-common-and-uncommon-columns"></a>搜索公共列和非公共列

```kusto
find Version == 'v1.0.0' or EventName == 'Event1' project Session_Id, EventText, Version, EventName
```

|source_|Session_Id|EventText|版本|EventName|
|---|---|---|---|---|
|EventsTable1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Some Text1|v1.0.0
|EventsTable1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Some Text2|v1.0.0
|EventsTable2|f7d5f95f-f580-4ea6-830b-5776c8d64fdd|Some Other Text1||Event1

注意：在实践中，将使用 ```Version == 'v1.0.0'``` 谓词筛选 EventsTable1 行，使用 ```EventName == 'Event1'``` 谓词筛选 EventsTable2 行 。

### <a name="use-abbreviated-notation-to-search-across-all-tables-in-the-current-database"></a>使用缩写表示法在当前数据库的所有表中进行搜索

```kusto
find Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e'
```

|source_|Session_Id|级别|EventText|pack_|
|---|---|---|---|---|
|EventsTable1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|信息|Some Text1|{"Version":"v1.0.0"}
|EventsTable1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|错误|Some Text2|{"Version":"v1.0.0"}
|EventsTable2|acbd207d-51aa-4df7-bfa7-be70eb68f04e|信息|Some Other Text2|{"EventName":"Event2"}
|EventsTable2|acbd207d-51aa-4df7-bfa7-be70eb68f04e|错误|Some Other Text3|{"EventName":"Event3"}


### <a name="return-the-results-from-each-row-as-a-property-bag"></a>以属性包的形式返回每一行的结果

```kusto
find Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e' project pack(*)
```

|source_|pack_|
|---|---|
|EventsTable1|{"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Information", "EventText":"Some Text1", "Version":"v1.0.0"}
|EventsTable1|{"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error", "EventText":"Some Text2", "Version":"v1.0.0"}
|EventsTable2|{"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Information", "EventText":"Some Other Text2", "EventName":"Event2"}
|EventsTable2|{"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error", "EventText":"Some Other Text3", "EventName":"Event3"}


## <a name="examples-of-cases-where-find-will-act-as-union"></a>将 `find` 用作 `union` 的案例示例

### <a name="using-a-non-tabular-expression-as-find-operand"></a>使用非表格表达式作为 find 操作数

```kusto
let PartialEventsTable1 = view() { EventsTable1 | where Level == 'Error' };
find in (PartialEventsTable1, EventsTable2) 
     where Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e'
```

### <a name="referencing-a-column-that-appears-in-multiple-tables-and-has-multiple-types"></a>引用出现在多个表中并具有多个类型的列

假设我们已通过运行以下内容创建了两个表： 

```kusto
.create tables 
  Table1 (Level:string, Timestamp:datetime, ProcessId:string),
  Table2 (Level:string, Timestamp:datetime, ProcessId:int64)
```

* 下面的查询将作为 `union` 执行。

```kusto
find in (Table1, Table2) where ProcessId == 1001
```

输出结果架构将为 (Level:string, Timestamp, ProcessId_string, ProcessId_int)。

* 以下查询也将作为 `union` 执行，但将产生不同的结果模式。

```kusto
find in (Table1, Table2) where ProcessId == 1001 project Level, Timestamp, ProcessId:string 
```

输出结果架构将为 (Level:string, Timestamp, ProcessId_string)
