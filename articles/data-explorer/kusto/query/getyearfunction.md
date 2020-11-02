---
title: getyear() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 getyear()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 5af6964c19bb8b2a291a60a19047a4301c04fbda
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103884"
---
# <a name="getyear"></a>getyear()

返回 `datetime` 参数的年份部分。

## <a name="example"></a>示例

```kusto
T
| extend year = getyear(datetime(2015-10-12))
// year == 2015
```