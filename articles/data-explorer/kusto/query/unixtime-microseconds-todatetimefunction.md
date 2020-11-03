---
title: unixtime_microseconds_todatetime() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 unixtime_microseconds_todatetime()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 11/27/2019
ms.date: 09/30/2020
ms.openlocfilehash: 57ba0b32649ddd2a3262847428ed590a31a9327a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106026"
---
# <a name="unixtime_microseconds_todatetime"></a>unixtime_microseconds_todatetime()

将 unix-epoch 微秒转换为 UTC 日期/时间。

## <a name="syntax"></a>语法

`unixtime_microseconds_todatetime(*microseconds*)`

## <a name="arguments"></a>参数

* microseconds：一个实数，表示以微秒为单位的纪元时间戳。 在纪元时间 (1970-01-01 00:00:00) 之前发生的 `Datetime` 时间戳值为负。

## <a name="returns"></a>返回

如果转换成功，则结果将为[日期/时间](./scalar-data-types/datetime.md)值。 如果转换不成功，则结果将为 null。

## <a name="see-also"></a>请参阅

* 使用 [unixtime_seconds_todatetime()](unixtime-seconds-todatetimefunction.md) 将 unix-epoch 秒转换为 UTC 日期/时间。
* 使用 [unixtime_milliseconds_todatetime()](unixtime-milliseconds-todatetimefunction.md) 将 unix-epoch 毫秒转换为 UTC 日期/时间。
* 使用 [unixtime_nanoseconds_todatetime()](unixtime-nanoseconds-todatetimefunction.md) 将 unix-epoch 纳秒转换为 UTC 日期/时间。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples  -->
```kusto
print date_time = unixtime_microseconds_todatetime(1546300800000000)
```

|date_time|
|---|
|2019-01-01 00:00:00.0000000|
