---
title: row_cumsum() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 row_cumsum()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 53024049340885043d78463108f8a1cb69c32de8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104888"
---
# <a name="row_cumsum"></a>row_cumsum()

计算[序列化行集](./windowsfunctions.md#serialized-row-set)中列的累计和。

## <a name="syntax"></a>语法

`row_cumsum` `(` *Term* [`,` *Restart* ] `)`

* “Term”是一个表达式，表示要求和的值。
  表达式必须是以下类型之一的标量：`decimal`、`int`、`long` 或 `real`。 Null Term 值不影响总和。
* Restart 是 `bool` 类型的可选参数，表示应何时重启累计运算（重新设置为 0）。 它可用于指示数据的分区；请参阅下面的第二个示例。

## <a name="returns"></a>返回

此函数返回其参数的累计和。

## <a name="examples"></a>示例

下面的示例演示如何计算前几个偶数整数的累计和。

```kusto
datatable (a:long) [
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10
]
| where a%2==0
| serialize cs=row_cumsum(a)
```

a    | cs
-----|-----
2    | 2
4    | 6
6    | 12
8    | 20
10   | 30

此示例演示了将数据分区（这里是指按 `name` 分区）时如何计算累计和（这里是指 `salary` 的累计和）：

```kusto
datatable (name:string, month:int, salary:long)
[
    "Alice", 1, 1000,
    "Bob",   1, 1000,
    "Alice", 2, 2000,
    "Bob",   2, 1950,
    "Alice", 3, 1400,
    "Bob",   3, 1450,
]
| order by name asc, month asc
| extend total=row_cumsum(salary, name != prev(name))
```

name   | 月份  | salary  | total
-------|--------|---------|------
Alice  | 1      | 1000    | 1000
Alice  | 2      | 2000    | 3000
Alice  | 3      | 1400    | 4400
Bob    | 1      | 1000    | 1000
Bob    | 2      | 1950    | 2950
Bob    | 3      | 1450    | 4400