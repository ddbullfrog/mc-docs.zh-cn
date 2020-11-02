---
title: hourofday() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 hourofday()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 44b7cc688a2967406968eb6791907a5397999784
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103872"
---
# <a name="hourofday"></a>hourofday()

返回一个整数，该整数表示给定日期的第几个小时

```kusto
hourofday(datetime(2015-12-14 18:54)) == 18
```

## <a name="syntax"></a>语法

`hourofday(`*a_date*`)`

## <a name="arguments"></a>参数

* `a_date`：`datetime`。

## <a name="returns"></a>返回

一天的 `hour number` (0-23)。