---
title: Datetime / timespan 算术 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 Datetime / timespan 算术。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 11/27/2019
ms.date: 10/29/2020
ms.openlocfilehash: 4a2cffa1236c0cef491565d0526afd0559c9d128
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104275"
---
# <a name="datetime--timespan-arithmetic"></a>Datetime / timespan 算术

Kusto 支持对 `datetime` 和 `timespan` 类型的值执行算术运算：

* 可以将两个 `datetime` 值相减（但不能相加）来得到一个表示二者之差的 `timespan` 值。
  例如，`datetime(1997-06-25) - datetime(1910-06-11)` 是 Jacques-Yves Cousteau 去世时的年龄。

* 可以将两个 `timespan` 值相加或相减以得到一个二者之和或差的 `timespan` 值。
  例如，`1d + 2d` 是三天。

* 可以在一个 `datetime` 值的基础上加上或减去一个 `timespan` 值。
  例如，`datetime(1910-06-11) + 1d` 是 Cousteau 年龄增大了一天的日期。

* 可以将两个 `timespan` 值相除，得到它们的商。
  例如，`1d / 5h` 的结果为 `4.8`。
  这样就可以将任何 `timespan` 值表示为另一个 `timespan` 值的倍数。 例如，若要以秒为单位表示一小时，只需将 `1h` 除以 `1s`：`1h / 1s`（结果很明显：`3600`）。

* 相反，一个数字值（如 `double` 和 `long`）可以乘以 `timespan` 值来得到 `timespan` 值。
  例如，可以将一个半小时表示为 `1.5 * 1h`。

## <a name="example-unix-time"></a>示例：Unix 时间

Unix 时间（也称为 POSIX 时间或 UNIX Epoch 时间）是一种系统，用于将某个时间点描述为自协调世界时 (UTC) 1970 年 1 月 1 日星期四 00:00:00 以来经过的秒数（减去闰秒）。

如果数据包括以整数形式表示的 Unix 时间，或者需要转换为这种时间，可使用以下函数：

```kusto
let fromUnixTime = (t:long)
{ 
    datetime(1970-01-01) + t * 1sec 
};
print result = fromUnixTime(1546897531)
```

|result                     |
|---------------------------|
|2019-01-07 21:45:31.0000000|

```kusto
let toUnixTime = (dt:datetime) 
{ 
    (dt - datetime(1970-01-01)) / 1s 
};
print result = toUnixTime(datetime(2019-01-07 21:45:31.0000000))
```

|result                     |
|---------------------------|
|1546897531                 |

> [!NOTE]
> 除上述函数外，请参阅用于 unix-epoch 时间转换的专用函数：[unixtime_seconds_todatetime()](unixtime-seconds-todatetimefunction.md)
> [unixtime_milliseconds_todatetime()](unixtime-milliseconds-todatetimefunction.md)
> [unixtime_microseconds_todatetime()](unixtime-microseconds-todatetimefunction.md)
> [unixtime_nanoseconds_todatetime()](unixtime-nanoseconds-todatetimefunction.md)