---
title: radians() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 radians()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 06ae5fc7063e79491a8e7d603f018f11974dc81f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104017"
---
# <a name="radians"></a>radians()

使用公式 `radians = (PI / 180 ) * angle_in_degrees`，将以度表示的角度值转换为以弧度表示的值

## <a name="syntax"></a>语法

`radians(`*a*`)`

## <a name="arguments"></a>参数

* a：以度表示的角度（实数）。

## <a name="returns"></a>返回

* 指定角度（度）的相应角度（弧度）。 

## <a name="examples"></a>示例

```kusto
print radians0 = radians(90), radians1 = radians(180), radians2 = radians(360) 

```

|radians0|radians1|radians2|
|---|---|---|
|1.5707963267949|3.14159265358979|6.28318530717959|