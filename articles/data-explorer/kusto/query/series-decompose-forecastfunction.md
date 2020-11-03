---
title: series_decompose_forecast() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 series_decompose_forecast()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 09/26/2019
ms.date: 09/30/2020
ms.openlocfilehash: 4db314087344fe7bf57e887de9beb6f34552e180
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106188"
---
# <a name="series_decompose_forecast"></a>series_decompose_forecast()

根据序列分解情况进行预测。

采用包含某个序列（动态数值数组）的表达式作为输入，并预测最后的尾随点的值。 有关详细信息，请参阅 [series_decompose](series-decomposefunction.md)。
 
## <a name="syntax"></a>语法

`series_decompose_forecast(`*Series* `,` *Points* `[,` *Seasonality*`,` *Trend*`,` *Seasonality_threshold*`])`

## <a name="arguments"></a>参数

* Series：数值的动态数组单元。 通常是 [make-series](make-seriesoperator.md) 或 [make_list](makelist-aggfunction.md) 运算符生成的输出。
* Points：一个整数，指定要预测的序列末端的点数。 这些点将从学习（回归）过程中排除。
* Seasonality：一个用于控制周期性分析的整数，包含以下项之一：
    * -1：使用 [series_periods_detect](series-periods-detectfunction.md) 自动检测周期性（默认值）。
    * period：一个正整数，按箱数指定所需期间。例如，如果将序列按 1 小时的箱分箱，则一个每周期间为 168 箱。
    * 0：无周期性（跳过提取此组件的操作）。
* Trend：一个用于控制趋势分析的字符串，包含以下项之一：
    * `linefit`：使用线性回归提取趋势组件（默认值）。
    * `avg`：将趋势组件定义为 average(x)。
    * `none`：无趋势，跳过提取此组件的操作。
* Seasonality_threshold：当 Seasonality 设置为 autodetect 时周期性分数的阈值。 默认分数阈值为 `0.6`。 有关详细信息，请参阅 [series_periods_detect](series-periods-detectfunction.md)。

**返回值**

 一个包含预测的序列的动态数组。

> [!NOTE]
> * 原始输入序列的动态数组应当包含许多要预测的点槽。 预测通常是通过使用 [make-series](make-seriesoperator.md) 并在包含要预测的时间范围的范围内指定结束时间来完成的。
> * 应当启用周期性或趋势，否则该函数将是冗余的，只返回用零填充的序列。

## <a name="example"></a>示例

在下面的示例中，我们将以小时粒度生成包含四周的一个序列，它具有每周周期性和小幅上升趋势。 然后，我们使用 `make-series` 并向序列中添加另一个空周。 `series_decompose_forecast` 按周（24*7 个点）调用，自动检测周期性和趋势，并生成整个五周期间的预测。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*4 step 1 // generate 4 weeks of hourly data
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| make-series y=max(y) on Timestamp in range(datetime(2018-03-01 05:00), datetime(2018-03-01 05:00)+24*7*5h, 1h); // create a time series of 5 weeks (last week is empty)
ts 
| extend y_forcasted = series_decompose_forecast(y, 24*7)  // forecast a week forward
| render timechart 
```

:::image type="content" source="images/series-decompose-forecastfunction/series-decompose-forecast.png" alt-text="序列分解预测":::
 
