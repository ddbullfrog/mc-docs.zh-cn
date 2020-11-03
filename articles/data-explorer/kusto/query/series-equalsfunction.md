---
title: series_equals() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_equals()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/01/2020
ms.date: 09/30/2020
ms.openlocfilehash: b008954b9ca838af742c0d08ee4b1ca6651d75cb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106184"
---
# <a name="series_equals"></a>series_equals()

计算两个数值序列输入的元素对应等于 (`==`) 逻辑运算。

## <a name="syntax"></a>语法

`series_equals (`*Series1*`,` *Series2*`)`

## <a name="arguments"></a>参数

* Series1、Series2：输入要进行元素对应比较的数值数组。 所有参数都必须是动态数组。 

## <a name="returns"></a>返回

布尔值的动态数组包括两个输入之间计算的元素对应相等逻辑运算。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 元素值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_equals_s2 = series_equals(s1, s2)
```

|s1|s2|s1_equals_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[false,true,false]|

## <a name="see-also"></a>请参阅

对于整个序列统计信息的比较，请参阅：
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
