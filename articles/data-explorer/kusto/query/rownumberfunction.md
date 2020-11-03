---
title: row_number() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 row_number()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: ebc347c7427268e845f49c4fdc46afb4bd9be84a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105804"
---
# <a name="row_number"></a>row_number()

返回[序列化行集](./windowsfunctions.md#serialized-row-set)中当前行的索引。
默认情况下，第一行的行索引从 `1` 开始，每增加一行就增加 `1`。
可以选择让行索引从不同于 `1` 的值开始。
此外，可以根据提供的谓词重置行索引。

## <a name="syntax"></a>语法

`row_number` `(` [ *StartingIndex* [`,` *Restart* ]] `)`

* StartingIndex 是 `long` 类型的常量表达式，表示要从其开始（或重新开始）的行索引的值。 默认值为 `1`。
* Restart 是 `bool` 类型的可选参数，表示何时将编号重新设置为 StartingIndex 值。 如果未提供，则使用默认值 `false`。

## <a name="returns"></a>返回

函数将当前行的行索引作为 `long` 类型的值返回。

## <a name="examples"></a>示例

以下示例返回一个包含两列的表，第一列 (`a`) 包含从 `10` 到 `1` 的数字，第二列 (`rn`) 包含从 `1` 到 `10` 的数字：

```kusto
range a from 1 to 10 step 1
| sort by a desc
| extend rn=row_number()
```

下面的示例与上面的示例类似，只是第二列 (`rn`) 从 `7` 开始：

```kusto
range a from 1 to 10 step 1
| sort by a desc
| extend rn=row_number(7)
```

最后一个示例展示了如何对数据进行分区，以及如何为每个分区的行编号。 此处，我们按 `Airport` 对数据进行分区：

```kusto
datatable (Airport:string, Airline:string, Departures:long)
[
  "TLV", "LH", 1,
  "TLV", "LY", 100,
  "SEA", "LH", 1,
  "SEA", "BA", 2,
  "SEA", "LY", 0
]
| sort by Airport asc, Departures desc
| extend Rank=row_number(1, prev(Airport) != Airport)
```

运行此查询会生成以下结果：

Airport  | 航空公司  | Departures  | 排名
---------|----------|-------------|------
SEA      | BA       | 2           | 1
SEA      | LH       | 1           | 2
SEA      | LY       | 0           | 3
TLV      | LY       | 100         | 1
TLV      | LH       | 1           | 2