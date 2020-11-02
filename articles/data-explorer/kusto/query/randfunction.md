---
title: rand() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 rand()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 4448d91f7e174aea7e65f0a0e2fcd9900f88bedd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104015"
---
# <a name="rand"></a>rand()

返回随机数。

```kusto
rand()
rand(1000)
```

## <a name="syntax"></a>语法

* `rand()` - 返回一个 `real` 类型的值，该值在 [0.0, 1.0) 范围内均匀分布。
* `rand(` N `)` - 返回从集 {0.0, 1.0, ..., N - 1} 中选择的 `real` 类型的均匀分布的值。