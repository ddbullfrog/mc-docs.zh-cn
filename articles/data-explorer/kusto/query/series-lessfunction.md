---
title: series_less() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_less()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/01/2020
ms.date: 09/30/2020
ms.openlocfilehash: 950ab0ccfc20b29773c558d53f4378844ea26829
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105379"
---
# <a name="series_less"></a>series_less()

计算两个数值序列输入的元素对应小于 (`<`) 逻辑运算。

## <a name="syntax"></a>语法

`series_less (`Series1`,` Series2`)`

## <a name="arguments"></a>参数

* Series1、Series2：输入要进行元素对应比较的数值阵列。 所有参数都必须是动态数组。 

## <a name="returns"></a>返回

布尔值的动态数组包括两个输入之间计算的元素对应小于逻辑运算。 任何非数值元素或非现有元素（不同大小的数组）会生成 `null` 元素值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_less_s2 = series_less(s1, s2)
```

|s1|s2|s1_less_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[true,false,false]|

## <a name="see-also"></a>请参阅

对于整个序列统计信息的比较，请参阅：
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
