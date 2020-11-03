---
title: schema_merge 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 schema_merge 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/16/2020
ms.date: 09/30/2020
ms.openlocfilehash: 86572b8c439c1ace2a5b966146358e4e61ef77c6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106198"
---
# <a name="schema_merge-plugin"></a>schema_merge 插件

将表格架构定义合并为统一架构。 

架构定义应采用 [`getschema`](./getschemaoperator.md) 运算符生成的格式。

`schema merge` 操作会联接输入架构中的列，并尝试将数据类型化简为常见的数据类型。 如果无法化简数据类型，则会在有问题的列上显示错误。

```kusto
let Schema1=Table1 | getschema;
let Schema2=Table2 | getschema;
union Schema1, Schema2 | evaluate schema_merge()
```

## <a name="syntax"></a>语法

`T` `|` `evaluate` `schema_merge(` *PreserveOrder* `)`

## <a name="arguments"></a>参数

* PreserveOrder：（可选）设置为 `true` 时，指示插件按保留的第一个表格架构的定义验证列顺序。 如果同一列采用多个架构，则列序号必须与该列显示时采用的第一个架构的列序号相同。 默认值为 `true`。

## <a name="returns"></a>返回

`schema_merge` 插件返回的输出与 [`getschema`](./getschemaoperator.md) 运算符返回的内容类似。

## <a name="examples"></a>示例

与追加了新列的架构合并。

```kusto
let schema1 = datatable(Uri:string, HttpStatus:int)[] | getschema;
let schema2 = datatable(Uri:string, HttpStatus:int, Referrer:string)[] | getschema;
union schema1, schema2 | evaluate schema_merge()
```

*结果*

|ColumnName | ColumnOrdinal | 数据类型 | ColumnType|
|---|---|---|---|
|Uri|0|System.String|string|
|HttpStatus|1|System.Int32|int|
|Referrer|2|System.String|string|

与列排序不同的架构合并（在新变体中，`HttpStatus` 序号从 `1` 更改为 `2`）。

```kusto
let schema1 = datatable(Uri:string, HttpStatus:int)[] | getschema;
let schema2 = datatable(Uri:string, Referrer:string, HttpStatus:int)[] | getschema;
union schema1, schema2 | evaluate schema_merge()
```

*结果*

|ColumnName | ColumnOrdinal | 数据类型 | ColumnType|
|---|---|---|---|
|Uri|0|System.String|string|
|Referrer|1|System.String|string|
|HttpStatus|-1|错误(CSL 类型未知:错误(列乱序))|错误(列乱序)|

与列排序不同但 `PreserveOrder` 设置为 `false` 的架构合并。

```kusto
let schema1 = datatable(Uri:string, HttpStatus:int)[] | getschema;
let schema2 = datatable(Uri:string, Referrer:string, HttpStatus:int)[] | getschema;
union schema1, schema2 | evaluate schema_merge(PreserveOrder = false)
```

*结果*

|ColumnName | ColumnOrdinal | 数据类型 | ColumnType|
|---|---|---|---|
|Uri|0|System.String|string
|Referrer|1|System.String|string
|HttpStatus|2|System.Int32|int|
