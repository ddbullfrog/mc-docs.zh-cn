---
title: datetime_part() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 datetime_part()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/18/2020
ms.date: 10/29/2020
ms.openlocfilehash: 366e09047fa1eb3b01662a46899f4cda5d700318
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103723"
---
# <a name="datetime_part"></a>datetime_part()

提取所请求的日期部分作为整数值。

```kusto
datetime_part("Day",datetime(2015-12-14))
```

## <a name="syntax"></a>语法

`datetime_part(`*part*`,`*datetime*`)`

## <a name="arguments"></a>参数

* `date`: `datetime`
* `part`: `string`

`part` 的可能值： 
* Year
* Quarter
* Month
* week_of_year
* 日期
* DayOfYear
* Hour
* Minute
* Second
* Millisecond
* Microsecond
* Nanosecond

## <a name="returns"></a>返回

一个表示已提取部分的整数。

> [!NOTE]
> `week_of_year` 返回一个表示周数的整数。 周数从一年的第一周算起，该周包括第一个周四。

## <a name="examples"></a>示例

```kusto
let dt = datetime(2017-10-30 01:02:03.7654321); 
print 
year = datetime_part("year", dt),
quarter = datetime_part("quarter", dt),
month = datetime_part("month", dt),
weekOfYear = datetime_part("week_of_year", dt),
day = datetime_part("day", dt),
dayOfYear = datetime_part("dayOfYear", dt),
hour = datetime_part("hour", dt),
minute = datetime_part("minute", dt),
second = datetime_part("second", dt),
millisecond = datetime_part("millisecond", dt),
microsecond = datetime_part("microsecond", dt),
nanosecond = datetime_part("nanosecond", dt)

```

|year|quarter|月份|weekOfYear|day|dayOfYear|hour|minute|第 2 个|毫秒|微秒|纳秒|
|---|---|---|---|---|---|---|---|---|---|---|---|
|2017|4|10|44|30|303|1|2|3|765|765432|765432100|

> [!NOTE]
> `weekofyear` 是 `week_of_year` 部分的已过时变体。 `weekofyear` 不符合 ISO 8601；一年的第一周被定义为一年中包含第一个周三的那一周。
`week_of_year` 符合 ISO 8601；一年的第一周被定义为一年中包含第一个周四的那一周。