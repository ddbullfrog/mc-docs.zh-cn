---
title: series_outliers() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_outliers()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/20/2019
ms.date: 09/30/2020
ms.openlocfilehash: cce00cf1ad96022d3c802cb91efe1f18854af700
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105373"
---
# <a name="series_outliers"></a>series_outliers()

对序列中的异常点进行评分。

该函数接受一个带有动态数值数组的表达式作为输入，生成一个相同长度的动态数值数组。 数组的每个值都表示使用 Tukey 测试时可能出现异常的分数。 相同输入元素中大于 1.5 的值表示异常增加， 小于 -1.5 的值表示异常减少。

## <a name="syntax"></a>语法

`series_outliers(`*x*`, `*kind*`, `*ignore_val*`, `*min_percentile*`, `*max_percentile*`)`

## <a name="arguments"></a>参数

* x：动态数组单元格，是数值数组
* *kind* ：离群值检测算法。 目前支持 `"tukey"`（传统“Tukey”）和 `"ctukey"`（自定义“Tukey”）。 默认为 `"ctukey"`
* ignore_val：一个数值，表示序列中缺少的值。 默认值为“double(null)”。 NULL 和忽略值的分数将设置为 `0`
* min_percentile：用于计算归一化四分位差。 默认值为 10，受支持的自定义值在 `[2.0, 98.0]` 范围内（仅限 `ctukey`）
* max_percentile：同样，默认值为 90，受支持的自定义值在 `[2.0, 98.0]` 范围内（仅限 ctukey）

下表描述了 `"tukey"` 和 `"ctukey"` 之间的差异：

| 算法 | 默认分位范围 | 支持自定义分位范围 |
|-----------|----------------------- |--------------------------------|
| `"tukey"` | 25% / 75%              | 否                             |
| `"ctukey"`| 10% / 90%              | 是                            |

> [!TIP]
> 使用此函数的最佳方式是将其应用于 [make-series](make-seriesoperator.md) 运算符的结果。

## <a name="example"></a>示例

带有某些干扰信息的时序会产生离群值。 若要将这些离群值（干扰信息）替换为平均值，请使用 series_outliers() 来检测出离群值，然后将其替换。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 100 step 1 
| extend y=iff(x==20 or x==80, 10*rand()+10+(50-x)/2, 10*rand()+10) // generate a sample series with outliers at x=20 and x=80
| summarize x=make_list(x),series=make_list(y)
| extend series_stats(series), outliers=series_outliers(series)
| mv-expand x to typeof(long), series to typeof(double), outliers to typeof(double)
| project x, series , outliers_removed=iff(outliers > 1.5 or outliers < -1.5, series_stats_series_avg , series ) // replace outliers with the average
| render linechart
``` 

:::image type="content" source="images/series-outliersfunction/series-outliers.png" alt-text="序列离群值" border="false":::
