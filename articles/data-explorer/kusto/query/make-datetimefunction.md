---
title: make_datetime() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 make_datetime()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: e64dd30f1a6c357e4b991b5372c2a3ab0fd08327
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105592"
---
# <a name="make_datetime"></a>make_datetime()

根据指定的日期和时间创建一个[日期/时间](./scalar-data-types/datetime.md)标量值。

```kusto
make_datetime(2017,10,01,12,10) == datetime(2017-10-01 12:10)
```

## <a name="syntax"></a>语法

`make_datetime(`*year* , *month* , *day*`)`

`make_datetime(`*year* , *month* , *day* , *hour* , *minute*`)`

`make_datetime(`*year* , *month* , *day* , *hour* , *minute* , *second*`)`

## <a name="arguments"></a>参数

* year：年（从 0 到 9999 的整数值）
* month：月（从 1 到 12 的整数值）
* day：日（从 1 到 28-31 的整数值）
* hour：小时（从 0 到 23 的整数值）
* minute：分钟（从 0 到 59 的整数值）
* second：秒（从 0 到 59.9999999 的实数值）

## <a name="returns"></a>返回

若创建成功，结果将是一个[日期/时间](./scalar-data-types/datetime.md)值，否则，结果将为 null。
 
## <a name="example"></a>示例

```kusto
print year_month_day = make_datetime(2017,10,01)
```

|year_month_day|
|---|
|2017-10-01 00:00:00.0000000|




```kusto
print year_month_day_hour_minute = make_datetime(2017,10,01,12,10)
```

|year_month_day_hour_minute|
|---|
|2017-10-01 12:10:00.0000000|




```kusto
print year_month_day_hour_minute_second = make_datetime(2017,10,01,12,11,0.1234567)
```

|year_month_day_hour_minute_second|
|---|
|2017-10-01 12:11:00.1234567|

