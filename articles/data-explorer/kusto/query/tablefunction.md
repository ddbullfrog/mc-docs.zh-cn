---
title: table()（范围函数）- Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 table()（范围函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: f54971654b8877e5f66d78805752a35f9a814f5b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104001"
---
# <a name="table-scope-function"></a>table()（范围函数）

table() 函数通过以 `string` 类型表达式的形式提供表的名称来引用表。

```kusto
table('StormEvent')
```

## <a name="syntax"></a>语法

`table` `(` *TableName* [`,` *DataScope* ] `)`

## <a name="arguments"></a>参数

::: zone pivot="azuredataexplorer"

* TableName：`string` 类型的表达式，提供要引用的表的名称。 在调用该函数时，此表达式的值必须是常数（即，它不会随数据上下文变化）。

* *DataScope* ：`string` 类型的可选参数，可用于根据数据按照表的有效 [缓存策略](../management/cachepolicy.md)归类的方式来限制表对此数据的引用。 如果使用该参数，则实际参数必须是具有以下可能值之一的常数 `string` 表达式：

    - `"hotcache"`：只有划分为热缓存类型的数据才会被引用。
    - `"all"`：该表中的所有数据都会被引用。
    - `"default"`：与 `"all"` 相同，群集管理员已将群集设置为使用 `"hotcache"` 作为默认值的情况除外。

::: zone-end

::: zone pivot="azuremonitor"

* TableName：`string` 类型的表达式，提供要引用的表的名称。 在调用该函数时，此表达式的值必须是常数（即，它不会随数据上下文变化）。

* *DataScope* ：`string` 类型的可选参数，可用于根据数据按照表的有效缓存策略归类的方式限制表对此数据的引用。 如果使用该参数，则实际参数必须是具有以下可能值之一的常数 `string` 表达式：

    - `"hotcache"`：只有划分为热缓存类型的数据才会被引用。
    - `"all"`：该表中的所有数据都会被引用。
    - `"default"`：这与 `"all"` 相同。

::: zone-end

## <a name="examples"></a>示例

### <a name="use-table-to-access-table-of-the-current-database"></a>使用 table() 访问当前数据库的表

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
table('StormEvent') | count
```

|计数|
|---|
|59066|

### <a name="use-table-inside-let-statements"></a>在 let 语句中使用 table()

可以重写与上述示例相同的查询，以使用接收参数 `tableName` 的内联函数（let 语句），该参数会传递到 table() 函数中。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let foo = (tableName:string)
{
    table(tableName) | count
};
foo('help')
```

|计数|
|---|
|59066|

### <a name="use-table-inside-functions"></a>在函数中使用 table()

可以重写与上述示例相同的查询，以便在一个接收参数 `tableName` 的函数中使用，该参数会传递到 table() 函数中。

```kusto
.create function foo(tableName:string)
{
    table(tableName) | count
};
```

::: zone pivot="azuredataexplorer"

**请注意：** 此类函数只能在本地使用，而不能在跨群集查询中使用。

::: zone-end

### <a name="use-table-with-non-constant-parameter"></a>将 table() 与非常数参数配合使用

不是标量常数字符串的参数不能作为参数传递到 `table()` 函数。

下面提供这种情况下的解决方法示例。

```kusto
let T1 = print x=1;
let T2 = print x=2;
let _choose = (_selector:string)
{
    union
    (T1 | where _selector == 'T1'),
    (T2 | where _selector == 'T2')
};
_choose('T2')

```

|x|
|---|
|2|
