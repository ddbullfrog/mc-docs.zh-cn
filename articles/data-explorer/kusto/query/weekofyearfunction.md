---
title: week_of_year() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 week_of_year()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/18/2020
ms.date: 09/30/2020
ms.openlocfilehash: 07cba68e2dd4376d34de40168a1c022e61155629
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104375"
---
# <a name="week_of_year"></a>week_of_year()

返回一个表示周数的整数。 根据 ISO 8601，周数从一年的第一周算起，该周包括第一个周四。

```kusto
week_of_year(datetime("2015-12-14"))
```

## <a name="syntax"></a>语法

`week_of_year(`*a_date*`)`

## <a name="arguments"></a>参数

* `a_date`：`datetime`。

## <a name="returns"></a>返回

`week number` - 包含给定日期的周数。

## <a name="examples"></a>示例

|输入                                    |输出|
|-----------------------------------------|------|
|`week_of_year(datetime(2020-12-31))`     |`53`  |
|`week_of_year(datetime(2020-06-15))`     |`25`  |
|`week_of_year(datetime(1970-01-01))`     |`1`   |
|`week_of_year(datetime(2000-01-01))`     |`52`  |

> [!NOTE]
> `weekofyear()` 是此函数的过时变体。 `weekofyear()` 不符合 ISO 8601；一年的第一周被定义为一年中包含第一个周三的那一周。
函数 `week_of_year()` 的当前版本符合 ISO 8601；一年的第一周被定义为一年中包含第一个周四的那一周。