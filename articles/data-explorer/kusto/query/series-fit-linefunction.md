---
title: series_fit_line() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_fit_line()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: a3ea3a4171e2ce9b299d301765dd0db1ecef56ab
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105910"
---
# <a name="series_fit_line"></a>series_fit_line()

对序列应用线性回归，返回多个列。  

采用包含动态数值数组作为输入的表达式，并执行线性回归以找出拟合度最好的那条线。 应对时间序列数组使用此函数，拟合 make-series 运算符的输出。 此函数生成以下列：
* `rsquare`：r-square 是用于衡量拟合质量的标准。 此值是 [0-1] 范围内的数字，其中 1 表示拟合度最好，0 表示数据无序，与任何直线均不拟合。 
* `slope`：近似直线的斜率（即 y=ax+b 中的“a”）。
* `variance`：输入数据的方差。
* `rvariance`：剩余方差，即输入数据值和近似数据值之间的方差。
* `interception`：近似直线的截距（即 y=ax+b 中的“b”）。
* `line_fit`：数值数组，其中包含拟合度最好的直线的一系列值。 序列长度等于输入数组的长度。 该值用于绘制图表。

## <a name="syntax"></a>语法

`series_fit_line(`*x*`)`

## <a name="arguments"></a>参数

* x：数值的动态数组。

> [!TIP]
> 使用此函数最便捷的方法是将其应用于 [make-series](make-seriesoperator.md) 运算符的结果。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([2,5,6,8,11,15,17,18,25,26,30,30])
| extend (RSquare,Slope,Variance,RVariance,Interception,LineFit)=series_fit_line(y)
| render timechart
```

:::image type="content" source="images/series-fit-line/series-fit-line.png" alt-text="序列拟合线":::

| RSquare | 斜率 | Variance | RVariance | Interception | LineFit                                                                                     |
|---------|-------|----------|-----------|--------------|---------------------------------------------------------------------------------------------|
| 0.982   | 2.730 | 98.628   | 1.686     | -1.666       | 1.064、3.7945、6.526、9.256、11.987、14.718、17.449、20.180、22.910、25.641、28.371、31.102 |
