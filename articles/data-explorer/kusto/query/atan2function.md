---
title: atan2() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 atan2()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: aab49f715d7399f73d3bc104e3941a8f219e6047
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106308"
---
# <a name="atan2"></a>atan2()

计算正 x 轴与从原点到点 (y, x) 的射线之间的角度（以弧度表示）。

## <a name="syntax"></a>语法

`atan2(`*y*`,`*x*`)`

## <a name="arguments"></a>参数

* x：X 坐标（实数）。
* y：Y 坐标（实数）。

## <a name="returns"></a>返回

* 正 x 轴与从原点到点 (y, x) 的射线之间的角度（以弧度表示）。

## <a name="examples"></a>示例

```kusto
print atan2_0 = atan2(1,1) // Pi / 4 radians (45 degrees)
| extend atan2_1 = atan2(0,-1) // Pi radians (180 degrees)
| extend atan2_2 = atan2(-1,0) // - Pi / 2 radians (-90 degrees)
```

|atan2_0|atan2_1|atan2_2|
|---|---|---|
|0.785398163397448|3.14159265358979|-1.5707963267949|