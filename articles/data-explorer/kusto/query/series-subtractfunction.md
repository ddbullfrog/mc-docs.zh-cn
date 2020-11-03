---
title: series_subtract() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_subtract()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: fbdedf2a5b94b77dd3676bee3959241cd918712b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105988"
---
# <a name="series_subtract"></a>series_subtract()

计算两个数值序列输入的元素对应减法。

## <a name="syntax"></a>语法

`series_subtract(`*series1*`,` *series2*`)`

## <a name="arguments"></a>参数

* series1、series2：输入数值数组，从第一个数组按元素减去第二个数组得到一个动态数组结果。 所有参数都必须是动态数组。 

## <a name="returns"></a>返回

在两个输入之间进行按元素减法运算计算出的动态数组。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 元素值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_subtract_s2 = series_subtract(s1, s2)
```

|s1|s2|s1_subtract_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[-3,0,3]|
|[2,4,8]|[8,4,2]|[-6,0,6]|
|[3,6,12]|[12,6,3]|[-9,0,9]|
