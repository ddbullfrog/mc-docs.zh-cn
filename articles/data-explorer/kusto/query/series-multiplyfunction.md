---
title: series_multiply() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_multiply()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: ad48dd6bb620b8a91c2af3d9ceeb9a120683a4b3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105989"
---
# <a name="series_multiply"></a>series_multiply()

计算两个数值序列输入的元素对应乘法。

## <a name="syntax"></a>语法

`series_multiply(`series1`,` series2`)` 

## <a name="arguments"></a>参数

* series1, series2：输入数值数组，按对应元素相乘获得动态数组结果。 所有参数都必须是动态数组。 

## <a name="returns"></a>返回

两个输入之间的元素对应乘法运算的计算结果的动态数组。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 元素值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_multiply_s2 = series_multiply(s1, s2)
```

|s1         |s2|        s1_multiply_s2|
|---|---|---|
|[1,2,4]    |[4,2,1]|   [4,4,4]|
|[2,4,8]    |[8,4,2]|   [16,16,16]|
|[3,6,12]   |[12,6,3]|  [36,36,36]|
