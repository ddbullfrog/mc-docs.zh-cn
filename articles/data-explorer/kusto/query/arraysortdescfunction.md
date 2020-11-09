---
title: array_sort_desc() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 array_sort_desc()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: slneimer
ms.service: data-explorer
ms.topic: reference
origin.date: 09/22/2020
ms.date: 10/30/2020
ms.openlocfilehash: be34f361175652608aab9c6e971ba54fde15df94
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106804"
---
# <a name="array_sort_desc"></a>array_sort_desc()

接收一个或多个数组。 按降序对第一个数组进行排序。 对其余数组进行排序，以匹配重新排序后的第一个数组。

## <a name="syntax"></a>语法

`array_sort_desc(`array1[, ..., argumentN]`)` 

`array_sort_desc(`array1[, ..., argumentN]`,`nulls_last`)`  

如果未提供 nulls_last，则使用默认值 `true`。

## <a name="arguments"></a>参数

* array1...arrayN：输入数组。
* nulls_last：一个布尔值，指示 `null` 是否应位于最后

## <a name="returns"></a>返回

返回与输入中相同数量的数组，第一个数组按升序排序，然后对其余数组排序，以匹配重新排序后的第一个数组。

对于与第一个数组长度不同的所有数组，均返回 `null`。

如果数组包含不同类型的元素，则按以下顺序对其进行排序：

* 数值、`datetime` 和 `timespan` 元素
* 字符串元素
* GUID 元素
* 其他所有元素

## <a name="example-1---sorting-two-arrays"></a>示例 1 - 对两个数组进行排序

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let array1 = dynamic([1,3,4,5,2]);
let array2 = dynamic(["a","b","c","d","e"]);
print array_sort_desc(array1,array2)
```

|`array1_sorted`|`array2_sorted`|
|---|---|
|[5,4,3,2,1]|["d","c","b","e","a"]|

> [!Note]
> 输出列名称根据函数的参数自动生成。 若要为输出列分配不同的名称，请使用以下语法：`... | extend (out1, out2) = array_sort_desc(array1,array2)`

## <a name="example-2---sorting-substrings"></a>示例 2 - 对子字符串进行排序

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let Names = "John,Paul,George,Ringo";
let SortedNames = strcat_array(array_sort_desc(split(Names, ",")), ",");
print result = SortedNames
```

|`result`|
|---|
|Ringo,Paul,John,George|

## <a name="example-3---combining-summarize-and-array_sort_desc"></a>示例 3 - 合并 summarize 和 array_sort_asc

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
datatable(command:string, command_time:datetime, user_id:string)
[
    'chmod',   datetime(2019-07-15),   "user1",
    'ls',      datetime(2019-07-02),   "user1",
    'dir',     datetime(2019-07-22),   "user1",
    'mkdir',   datetime(2019-07-14),   "user1",
    'rm',      datetime(2019-07-27),   "user1",
    'pwd',     datetime(2019-07-25),   "user1",
    'rm',      datetime(2019-07-23),   "user2",
    'pwd',     datetime(2019-07-25),   "user2",
]
| summarize timestamps = make_list(command_time), commands = make_list(command) by user_id
| project user_id, commands_in_chronological_order = array_sort_desc(timestamps, commands)[1]
```

|`user_id`|`commands_in_chronological_order`|
|---|---|
|user1|[<br>  "rm",<br>  "pwd",<br>  "dir",<br>  "chmod",<br>  "mkdir",<br>  "ls"<br>]|
|user2|[<br>  "pwd",<br>  "rm"<br>]|

> [!Note]
> 如果数据可能包含 `null` 值，请使用 [make_list_with_nulls](make-list-with-nulls-aggfunction.md)，而不是 [make_list](makelist-aggfunction.md)。

## <a name="example-4---controlling-location-of-null-values"></a>示例 4 - 控制 `null` 值的位置

默认情况下，`null` 值放在已排序数组的最后。 但是，你可以显式控制它，方法是将 `bool` 值添加为 `array_sort_desc()` 的最后一个参数。

具有默认行为的示例：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print array_sort_desc(dynamic([null,"blue","yellow","green",null]))
```

|`print_0`|
|---|
|["yellow","green","blue",null,null]|

具有非默认行为的示例：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print array_sort_desc(dynamic([null,"blue","yellow","green",null]), false)
```

|`print_0`|
|---|
|[null,null,"yellow","green","blue"]|

## <a name="see-also"></a>请参阅

若要按升序对第一个数组排序，请使用 [array_sort_asc()](arraysortascfunction.md)。
