---
title: datetime 数据类型 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 datetime 数据类型。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: 9c98734452b84ef27bda3302a9c8a77519e94f00
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104568"
---
# <a name="the-datetime-data-type"></a>datetime 数据类型

`datetime` (`date`) 数据类型表示某个时刻，通常以日期和当天的时间表示。
数值范围为从公历的公元 0001 年 1 月 1 日 00:00:00（午夜）到公元 9999 年 12 月 31 日 晚上 11:59:59。 

时间值以 100 纳秒为单位（称为时钟周期）进行度量。某个特定日期是指 GregorianCalendar 日历中自公元 0001 年 1 月 1 日午夜 12:00 以来的 时钟周期数（不包括因考虑到闰秒而添加的时钟周期）。
例如，时钟周期值 31241376000000000 表示 0100 年 1 月 1 日（星期五）午夜 12:00:00。
这有时称为“线性时间点”。

> [!WARNING]
> Kusto 中的 `datetime` 值始终在 UTC 时区中。 在其他时区中显示 `datetime` 值是显示数据的用户应用程序的职责，而不是数据本身的属性。 如果需要将时区值保存为数据的一部分，则应使用单独的列（提供相对于 UTC 的偏移信息）。

## <a name="datetime-literals"></a>datetime 文本

`datetime` 类型的文本采用语法为 `datetime(`value`)`，其中的 value 支持多种格式，如下表所示：

|示例                                                     |值                                                         |
|------------------------------------------------------------|--------------------------------------------------------------|
|`datetime(2015-12-31 23:59:59.9)`<br/>`datetime(2015-12-31)`|时间始终采用 UTC。 省略日期，提供当天时间。|
|`datetime(null)`                                            |请参阅 [NULL 值](null-values.md)。                            |
|`now()`                                                     |当前时间。                                             |
|`now(`-*timespan*`)`                                        |`now()-`*timespan*                                            |
|`ago(`*timespan*`)`                                         |`now()-`*timespan*                                            |

`now()` 和 `ago()` 表示与 Kusto 开始执行查询时的时间点进行比较的 `datetime` 值。 这些可以在同一查询中出现多次，并且所有这些都将使用一个值。
（换句话说，诸如 `now(-x) - ago(x)` 之类的表达式的求值始终为 `timespan` 值 0。）

## <a name="supported-formats"></a>支持的格式

`datetime` 有多种格式，这些格式可以用作 [datetime() 文本](#datetime-literals)和 [todatetime()](../todatetimefunction.md) 函数。

> [!WARNING]
> 强烈建议只使用 ISO 8601 格式。

### <a name="iso-8601"></a>[ISO 8601](https://www.iso.org/iso/home/standards/iso8601.htm)

|格式|示例|
|------|-------|
|%Y-%m-%dT%H:%M:%s%z|2014-05-25T08:20:03.123456Z|
|%Y-%m-%dT%H:%M:%s|2014-05-25T08:20:03.123456|
|%Y-%m-%dT%H:%M|2014-05-25T08:20|
|%Y-%m-%d %H:%M:%s%z|2014-11-08 15:55:55.123456Z|
|%Y-%m-%d %H:%M:%s|2014-11-08 15:55:55|
|%Y-%m-%d %H:%M|2014-11-08 15:55|
|%Y-%m-%d|2014-11-08|

### <a name="rfc-822"></a>[RFC 822](https://www.ietf.org/rfc/rfc0822.txt)

|格式|示例|
|------|-------|
|%w, %e %b %r %H:%M:%s %Z|Sat, 8 Nov 14 15:05:02 GMT|
|%w, %e %b %r %H:%M:%s|Sat, 8 Nov 14 15:05:02|
|%w, %e %b %r %H:%M|Sat, 8 Nov 14 15:05|
|%w, %e %b %r %H:%M %Z|Sat, 8 Nov 14 15:05 GMT|
|%e %b %r %H:%M:%s %Z|8 Nov 14 15:05:02 GMT|
|%e %b %r %H:%M:%s|8 Nov 14 15:05:02|
|%e %b %r %H:%M|8 Nov 14 15:05|
|%e %b %r %H:%M %Z|8 Nov 14 15:05 GMT|

### <a name="rfc-850"></a>[RFC 850](https://tools.ietf.org/html/rfc850)

|格式|示例|
|------|-------|
|%w, %e-%b-%r %H:%M:%s %Z|Saturday, 08-Nov-14 15:05:02 GMT|
|%w, %e-%b-%r %H:%M:%s|Saturday, 08-Nov-14 15:05:02|
|%w, %e-%b-%r %H:%M %Z|Saturday, 08-Nov-14 15:05 GMT|
|%w, %e-%b-%r %H:%M|Saturday, 08-Nov-14 15:05|
|%e-%b-%r %H:%M:%s %Z|08-Nov-14 15:05:02 GMT|
|%e-%b-%r %H:%M:%s|08-Nov-14 15:05:02|
|%e-%b-%r %H:%M %Z|08-Nov-14 15:05 GMT|
|%e-%b-%r %H:%M|08-Nov-14 15:05|


### <a name="sortable"></a>可排序 

|格式|示例|
|------|-------|        
|%Y-%n-%e %H:%M:%s|2014-11-08 15:05:25|
|%Y-%n-%e %H:%M:%s %Z|2014-11-08 15:05:25 GMT|
|%Y-%n-%e %H:%M|2014-11-08 15:05|
|%Y-%n-%e %H:%M %Z|2014-11-08 15:05 GMT|
|%Y-%n-%eT%H:%M:%s|2014-11-08T15:05:25|
|%Y-%n-%eT%H:%M:%s %Z|2014-11-08T15:05:25 GMT|
|%Y-%n-%eT%H:%M|2014-11-08T15:05|
|%Y-%n-%eT%H:%M %Z|2014-11-08T15:05 GMT|
