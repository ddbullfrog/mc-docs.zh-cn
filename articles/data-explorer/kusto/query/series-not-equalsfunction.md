---
title: series_not_equals() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_not_equals()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/01/2020
ms.date: 09/30/2020
ms.openlocfilehash: 054f77d3670107293166fc28ced470baf84b9051
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105374"
---
# <a name="series_not_equals"></a>series_not_equals()

计算两个数值序列输入的元素对应不等于 (`!=`) 逻辑运算。

## <a name="syntax"></a>语法

`series_not_equals (`*Series1*`,` *Series2*`)`

## <a name="arguments"></a>参数

* Series1、Series2：输入要进行元素对应比较的数值数组。 所有参数都必须是动态数组。 

## <a name="returns"></a>返回

布尔值的动态数组，包括两个输入之间计算的元素对应不等逻辑运算。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 元素值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_not_equals_s2 = series_not_equals(s1, s2)
```

|s1|s2|s1_not_equals_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[true,false,true]|

## <a name="see-also"></a>请参阅

若要进行整个序列统计信息的比较，请参阅：
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
