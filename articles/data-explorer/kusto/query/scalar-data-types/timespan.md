---
title: timespan 数据类型 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 timespan 数据类型。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 8602fcfd973b1c7370ad6023bd405e025b0dd12b
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146690"
---
# <a name="the-timespan-data-type"></a>timespan 数据类型

`timespan` (`time`) 数据类型表示时间间隔。

## <a name="timespan-literals"></a>timespan 文本

`timespan` 类型的文本采用语法为 `timespan(`value`)`，其中的 value 支持多种格式，如下表所示：

|Value|时间长度|
---|---
`2d`|2 天
`1.5h`|1.5 小时
`30m`|30 分钟
`10s`|10 秒
`0.1s`|0.1 秒
`100ms`| 100 毫秒
`10microsecond`|10 微秒
`1tick`|100 纳秒
`time(15 seconds)`|15 秒
`time(2)`| 2 天
`time(0.12:34:56.7)`|`0d+12h+34m+56.7s`

特殊形式 `time(null)` 是 [null 值](null-values.md)。

## <a name="timespan-operators"></a>timespan 运算符

两个 `timespan` 类型的值可以相加、相减和相除。
最后一个操作返回一个 `real` 类型的值，表示一个值可以是另一个值的小数倍。

## <a name="examples"></a>示例

下面的示例以多种方式计算一天中的秒数：

```kusto
print
    result1 = 1d / 1s,
    result2 = time(1d) / time(1s),
    result3 = 24 * 60 * time(00:01:00) / time(1s)
```