---
title: series_fit_2lines_dynamic() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 series_fit_2lines_dynamic()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 24946b0117f9541349e38c39aca2ebd63e3aa31e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105509"
---
# <a name="series_fit_2lines_dynamic"></a>series_fit_2lines_dynamic()

对序列应用两段线性回归，返回动态对象。  

采用包含动态数值数组的表达式作为输入，并应用两段线性回归，以便识别和量化序列中的趋势变化。 该函数针对序列索引进行迭代。 在每次迭代中，它将序列拆分为两部分，并使用 [series_fit_line()](series-fit-linefunction.md) 或 [series_fit_line_dynamic()](series-fit-line-dynamicfunction.md) 来拟合一个单独的线条。 此函数将线条与两个部分中的每部分进行拟合，并计算总的 R 平方值。 最佳拆分是最大化 R 平方的拆分。 函数以动态值返回包含以下内容的参数：

* `rsquare`：R 平方是用于衡量拟合质量的标准。 它是 [0-1] 范围内的一个数字，其中 1 表示拟合质量可能最佳，0 表示数据是无序的，无法与任何线拟合。
* `split_idx`：两段的拆分点的索引（从零开始）。
* `variance`：输入数据的方差。
* `rvariance`：残差方差，即输入数据值和近似数据值之间的方差（按两个线段）。
* `line_fit`：数值数组，其中包含拟合度最好的直线的一系列值。 序列长度等于输入数组的长度。 它用于绘制图表。
* `right.rsquare`：拆分点右侧直线的相关系数平方，请参阅 [series_fit_line()](series-fit-linefunction.md) 或 [series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)。
* `right.slope`：右近似线（形式为 y=ax+b）的斜率。
* `right.interception`：左近似线的截距（即 y=ax+b 中的 b）。
* `right.variance`：拆分点右侧输入数据的方差。
* `right.rvariance`：拆分点右侧输入数据的残差方差。
* `left.rsquare`：拆分点左侧直线的相关系数平方，请参阅 [series_fit_line()].(series-fit-linefunction.md) 或 [series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)。
* `left.slope`：左近似线（形式为 y=ax+b）的斜率。
* `left.interception`：左近似线（形式为 y=ax+b）的截距。
* `left.variance`：拆分点左侧输入数据的方差。
* `left.rvariance`：拆分点左侧输入数据的残差方差。

此运算符类似于 [series_fit_2lines](series-fit-2linesfunction.md)。 与 `series-fit-2lines` 不同，它返回一个动态包。

## <a name="syntax"></a>语法

`series_fit_2lines_dynamic(`*x*`)`

## <a name="arguments"></a>参数

* x：数值的动态数组。  

> [!TIP]
> 使用此函数最便捷的方法是将其应用于 [make-series](make-seriesoperator.md) 运算符的结果。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend LineFit=series_fit_line_dynamic(y).line_fit, LineFit2=series_fit_2lines_dynamic(y).line_fit
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/series-fit-2lines/series-fit-2lines.png" alt-text="序列拟合 2 条线":::
