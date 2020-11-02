---
title: series_fit_2lines() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_fit_2lines()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 6877946943c754d9f977baa5cb21b821833bf819
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105498"
---
# <a name="series_fit_2lines"></a>series_fit_2lines()

对一个序列应用两段线性回归，返回多个列。  

采用包含动态数值数组的表达式作为输入，并应用两段线性回归，以便识别和量化序列中的趋势变化。 该函数针对序列索引进行迭代。 在每次迭代中，该函数将序列拆分为两部分，为每个部分单独拟合一条线（使用 [series_fit_line()](series-fit-linefunction.md)），并计算总相关系数平方。 最好的拆分方式是最大限度提高相关系数平方值；函数将返回其参数：


|参数  |描述  |
|---------|---------|
|`rsquare`     | 相关系数平方是衡量拟合质量的标准。 它是 [0-1] 范围内的数字，其中 1 表示拟合质量可能最佳，0 表示数据是无序的且无法与任何线拟合。        |
|`split_idx`     |   两段的拆分点的索引（从零开始）。      |
|`variance`     | 输入数据的方差。        |
|`rvariance`     | 剩余方差，即输入数据值和近似数据值之间的方差（按两个线段）。        |
|`line_fit`     | 包含一系列最佳拟合线值的数值数组。 序列长度等于输入数组的长度。 它主要用于绘制图表。        |
|`right_rsquare`     | 拆分点右侧线的相关系数平方，请参阅 [series_fit_line()](series-fit-linefunction.md)。        |
|`right_slope`     | 右近似线（形式为 y=ax+b）的斜率。         |
|`right_interception`     |  左近似线的截距（即 y=ax+b 中的 b）。       |
|`right_variance`    | 拆分点右侧输入数据的方差。        |
|`right_rvariance`     | 拆分点右侧输入数据的剩余方差。        |
|`left_rsquare`     | 拆分点左侧线的相关系数平方，请参阅 [series_fit_line()](series-fit-linefunction.md)。        |
|`left_slope`    | 左近似线（形式为 y=ax+b）的斜率。        |
|`left_interception`     |   左近似线（形式为 y=ax+b）的截距。      |
|`left_variance`     | 拆分点左侧输入数据的方差。        |
|`left_rvariance`     | 拆分点左侧输入数据的剩余方差。        |


> [!Note]
> 此函数返回多个列，因此不能用作另一个函数的参数。

## <a name="syntax"></a>语法

project `series_fit_2lines(`*x*`)`
* 将会返回上述所有列，其名称为：series_fit_2lines_x_rsquare、series_fit_2lines_x_split_idx 等。

project (rs, si, v)=`series_fit_2lines(`*x*`)`
* 将会返回以下列：rs（相关系数平方）、si（拆分索引）、v（方差），其余列的名称将会类似于 series_fit_2lines_x_rvariance、series_fit_2lines_x_line_fit 等。

extend (rs, si, v)=`series_fit_2lines(`*x*`)`
* 将仅返回：rs（相关系数平方）、si（拆分索引）和 v（方差）。
  
## <a name="arguments"></a>参数

* x：数值的动态数组。  

> [!TIP]
> 使用此函数最便捷的方法是将其应用于 [make-series](make-seriesoperator.md) 运算符的结果。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend (Slope,Interception,RSquare,Variance,RVariance,LineFit)=series_fit_line(y), (RSquare2, SplitIdx, Variance2,RVariance2,LineFit2)=series_fit_2lines(y)
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/series-fit-2lines/series-fit-2lines.png" alt-text="序列拟合 2 条线":::
