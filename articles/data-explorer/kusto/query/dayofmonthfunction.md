---
title: dayofmonth() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 dayofmonth()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 9dae274a71c7b4404232003e339464681b6cf491
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104627"
---
# <a name="dayofmonth"></a>dayofmonth()

返回一个整数，该整数表示给定月的第几天

```kusto
dayofmonth(datetime(2015-12-14)) == 14
```

## <a name="syntax"></a>语法

`dayofmonth(`*a_date*`)`

## <a name="arguments"></a>参数

* `a_date`：`datetime`。

## <a name="returns"></a>返回

给定月份的 `day number`。