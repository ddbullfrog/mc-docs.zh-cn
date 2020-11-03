---
title: series_fit_line_dynamic() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_fit_line_dynamic()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 792fc5f5206338c5a19b214538a09e0ea18b69d0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105911"
---
# <a name="series_fit_line_dynamic"></a>series_fit_line_dynamic()

对序列应用线性回归，返回动态对象。  

采用包含动态数值数组作为输入的表达式，并执行线性回归以找出拟合度最好的那条线。 应对时间序列数组使用此函数，拟合 make-series 运算符的输出。 它生成具有以下内容的动态值：
* `rsquare`：r-square 是用于衡量拟合质量的标准。 它是 [0-1] 范围内的数字，其中 1 表示拟合度最好，0 表示数据无序，与任何直线均不拟合
* `slope`：近似直线的斜率（即 y=ax+b 中的 a 值）
* `variance`：输入数据的方差
* `rvariance`：剩余方差，即输入数据值和近似数据值之间的方差。
* `interception`：近似直线的截距（即 y=ax+b 中的 b 值）
* `line_fit`：数值数组，其中包含拟合度最好的直线的一系列值。 序列长度等于输入数组的长度。 它主要用于绘制图表。

此运算符与 [series_fit_line](series-fit-linefunction.md) 相似，但与 `series-fit-line` 不同（它返回动态包）。

## <a name="syntax"></a>语法

`series_fit_line_dynamic(`*x*`)`

## <a name="arguments"></a>参数

* x：数值的动态数组。

> [!TIP]
> 使用此函数最便捷的方法是将其应用于 [make-series](make-seriesoperator.md) 运算符的结果。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([2,5,6,8,11,15,17,18,25,26,30,30])
| extend fit=series_fit_line_dynamic(y)
| extend RSquare=fit.rsquare, Slope=fit.slope, Variance=fit.variance,RVariance=fit.rvariance,Interception=fit.interception,LineFit=fit.line_fit
| render timechart
```
 
:::image type="content" source="images/series-fit-line/series-fit-line.png" alt-text="序列拟合线":::

| RSquare | 斜率 | Variance | RVariance | Interception | LineFit                                                                                     |
|---------|-------|----------|-----------|--------------|---------------------------------------------------------------------------------------------|
| 0.982   | 2.730 | 98.628   | 1.686     | -1.666       | 1.064、3.7945、6.526、9.256、11.987、14.718、17.449、20.180、22.910、25.641、28.371、31.102 |
 
