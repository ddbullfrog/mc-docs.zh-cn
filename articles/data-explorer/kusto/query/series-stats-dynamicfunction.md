---
title: series_stats_dynamic() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_stats_dynamic()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/10/2020
ms.date: 09/30/2020
ms.openlocfilehash: 99c361c58df7a2b7f4006e0c5795625ad0bb952c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105365"
---
# <a name="series_stats_dynamic"></a>series_stats_dynamic()

返回动态对象中的序列的统计信息。  

`series_stats_dynamic()` 函数接受包含动态数值数组的列作为输入，并生成具有以下内容的动态值：
* `min`：输入数组中的最小值
* `min_idx`：输入数组中的最小值的第一个位置
* `max`：输入数组中的最大值
* `max_idx`：输入数组中的最大值的第一个位置
* `avg`：输入数组的平均值
* `variance`：输入数组的样本方差
* `stdev`：输入数组的样本标准偏差

## <a name="syntax"></a>语法

`series_stats_dynamic(`x `[,`ignore_nonfinite`])` 

## <a name="arguments"></a>参数

* x：动态数组单元格（数值数组）。 
* ignore_nonfinite：布尔值（可选，默认值：`false`）标志，该标志指定是否在计算统计信息的同时忽略非有限值（null、NaN、inf 等）。 如果设置为 `false`，则返回的结果为 `null`（如果数组中存在非有限值）。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print x=dynamic([23,46,23,87,4,8,3,75,2,56,13,75,32,16,29]) 
| project stats=series_stats_dynamic(x)
```

|stats
|---|
|{"min":2.0, "min_idx":8, "max":87.0, "max_idx":3, "avg":32.8, "stdev":28.503633853548269, "variance":812.45714285714291 }
