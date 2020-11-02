---
title: dayofweek() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 dayofweek()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: f5e453ce4f4e95e91d442cd620629f6723399d6e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104807"
---
# <a name="dayofweek"></a>dayofweek()

返回自上个星期日以来的整数天数，用作 `timespan`。

```kusto
dayofweek(datetime(2015-12-14)) == 1d  // Monday
```

## <a name="syntax"></a>语法

`dayofweek(`*a_date*`)`

## <a name="arguments"></a>参数

* `a_date`：`datetime`。

## <a name="returns"></a>返回

`timespan`从上个星期日的午夜开始，向下舍入为整数天数。

## <a name="examples"></a>示例

```kusto
dayofweek(datetime(1947-11-30 10:00:05))  // time(0.00:00:00), indicating Sunday
dayofweek(datetime(1970-05-11))           // time(1.00:00:00), indicating Monday
```
