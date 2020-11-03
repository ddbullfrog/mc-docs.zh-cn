---
title: bin_at() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 bin_at()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: fea76b336228b880c2b17ec4f43a0ea738c8e558
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106047"
---
# <a name="bin_at"></a>bin_at()

将值向下舍入为固定大小的“bin”，并控制 bin 的起点。
（另请参阅 [`bin function`](./binfunction.md)。）

## <a name="syntax"></a>语法

`bin_at` `(`*Expression*`,` *BinSize*`, ` *FixedPoint*`)`

## <a name="arguments"></a>参数

* *表达式* ：数值类型（包括 `datetime` 和 `timespan`）的标量表达式，指示要舍入的值。
* BinSize：数值类型或 `timespan` 的标量常量（适用于 `datetime` 或 `timespan` 表达式），指示每个箱的大小。
* FixedPoint：与 Expression 类型相同的标量常数，指示 Expression 的一个值，该值是一个“固定点”（即 `bin_at(fixed_point, bin_size, fixed_point) == fixed_point` 的一个值 `fixed_point`） 。

## <a name="returns"></a>返回

Expression 下方最接近的 BinSize 倍数，移动后 FixedPoint 将被转换为自身  。

## <a name="examples"></a>示例

|表达式                                                                    |结果                           |注释                   |
|------------------------------------------------------------------------------|---------------------------------|---------------------------|
|`bin_at(6.5, 2.5, 7)`                                                         |`4.5`                            ||
|`bin_at(time(1h), 1d, 12h)`                                                   |`-12h`                           ||
|`bin_at(datetime(2017-05-15 10:20:00.0), 1d, datetime(1970-01-01 12:00:00.0))`|`datetime(2017-05-14 12:00:00.0)`|所有 bin 都将处于中午   |
|`bin_at(datetime(2017-05-17 10:20:00.0), 7d, datetime(2017-06-04 00:00:00.0))`|`datetime(2017-05-14 00:00:00.0)`|所有 bin 都将在星期日|


在下面的示例中，注意 `"fixed point"` 参数将作为一个 bin 返回，其他 bin 根据 `bin_size` 与其对齐。 另请注意，每个日期/时间 bin 表示该 bin 的开始时间：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto

datatable(Date:datetime, Num:int)[
datetime(2018-02-24T15:14),3,
datetime(2018-02-23T16:14),4,
datetime(2018-02-26T15:14),5]
| summarize sum(Num) by bin_at(Date, 1d, datetime(2018-02-24 15:14:00.0000000)) 
```

|Date|sum_Num|
|---|---|
|2018-02-23 15:14:00.0000000|4|
|2018-02-24 15:14:00.0000000|3|
|2018-02-26 15:14:00.0000000|5|
