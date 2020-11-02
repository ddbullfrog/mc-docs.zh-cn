---
title: degrees() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 degrees()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 89e4279496b09857f322d5eb71fb0f6a591d40b6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104806"
---
# <a name="degrees"></a>degrees()

使用公式 `degrees = (180 / PI ) * angle_in_radians`，将以弧度表示的角度值转换为以度表示的值

## <a name="syntax"></a>语法

`degrees(`*a*`)`

## <a name="arguments"></a>参数

* a：以弧度表示的角度（实数）。

## <a name="returns"></a>返回

* 指定角度（弧度）的相应角度（度）。 

## <a name="examples"></a>示例

```kusto
print degrees0 = degrees(pi()/4), degrees1 = degrees(pi()*1.5), degrees2 = degrees(0)

```

|degrees0|degrees1|degrees2|
|---|---|---|
|45|270|0|
