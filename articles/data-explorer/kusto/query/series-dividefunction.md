---
title: series_divide() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_divide()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: d1ebce7efaa3ffa216e565505a97455f9f1f6884
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106185"
---
# <a name="series_divide"></a>series_divide()

计算两个数值序列输入的元素对应除法。

## <a name="syntax"></a>语法

`series_divide(`*series1*`,` *series2*`)`

## <a name="arguments"></a>参数

* series1、series2：输入数值数组，第一个数组按元素除以第二个数组得到一个动态数组结果。 所有参数都必须是动态数组。 

## <a name="returns"></a>返回

在两个输入之间进行按元素除法运算计算出的动态数组。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 元素值。

请注意：结果系列的类型为 double，即使输入是整数也是如此。 除以零与 double 类型的除以零遵循相同的规则（例如，2/0 会生成 double(+inf)）。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_divide_s2 = series_divide(s1, s2)
```

|s1         |s2|        s1_divide_s2|
|---|---|---|
|[1,2,4]    |[4,2,1]|   [0.25,1.0,4.0]|
|[2,4,8]    |[8,4,2]|   [0.25,1.0,4.0]|
|[3,6,12]   |[12,6,3]|  [0.25,1.0,4.0]|
