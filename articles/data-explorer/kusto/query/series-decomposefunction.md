---
title: series_decompose() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 series_decompose()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 09/26/2019
ms.date: 09/30/2020
ms.openlocfilehash: 1c5cc530802ef3840b1b5ed8296aa5384ce06caf
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106186"
---
# <a name="series_decompose"></a>series_decompose()

对序列应用分解转换。  

采用包含序列（动态数值数组）的表达式作为输入，并将其分解为周期性、趋势和残差组件。
 
## <a name="syntax"></a>语法

`series_decompose(`*Series* `[,` *Seasonality*`,` *Trend*`,` *Test_points*`,` *Seasonality_threshold*`])`

## <a name="arguments"></a>参数

* Series：动态数组单元格（数值数组），通常是 [make-series](make-seriesoperator.md) 或 [make_list](makelist-aggfunction.md) 运算符生成的输出
* Seasonality：一个用于控制周期性分析的整数，其中包含
    * -1：使用 [series_periods_detect](series-periods-detectfunction.md) 自动检测周期性（默认值）。
    * period：一个正整数，按箱数指定所需期间。例如，如果将序列按 1 小时的箱分箱，则一个每周期间为 168 箱。
    * 0：无周期性（跳过提取此组件的操作）。    
* Trend：一个用于控制趋势分析的字符串，包含下列值之一：
    * “avg”：将趋势组件定义为 average(x)（默认值）
    * “linefit”：使用线性回归提取趋势组件。
    * “none”：无趋势，跳过提取此组件的操作。    
* Test_points：0（默认值）或正整数，指定要从学习（回归）过程中排除的序列末尾处的点的数目。 应当设置此参数，用于预测。
* Seasonality_threshold：当 Seasonality 设置为 autodetect 时周期性分数的阈值，默认分数阈值为 `0.6`。 有关详细信息，请参阅 [series_periods_detect](series-periods-detectfunction.md)。

**返回值**

 该函数返回以下各个序列：

* `baseline`：序列的预测值（周期性组件和趋势组件的总和，见下）。
* `seasonal`：周期性组件的序列：
    * 如果未检测到期间或者它显式设置为 0，则为常数 0。
    * 如果检测到期间或者它设置为正整数，则为同一阶段中序列点的中值
* `trend`：趋势组件的序列。
* `residual`：残差组件的序列（即 x - 基线）。
  

**备注**

* 组件执行顺序：
    1. 提取周期性序列
    2. 从 x 中减去它，生成去除周期性变动的序列
    3. 从去除周期性变动的序列中提取趋势组件
    4. 创建基线 = 周期性 + 趋势
    5. 创建残差 = x - 基线
    
* 应当启用周期性和/或趋势。 否则，该函数是冗余的，只返回“基线 = 0”和“残差 = x”。

**有关序列分解的更多信息**

此方法通常应用于预计会表现出周期性和/或趋势行为的指标的时序。 你可以使用此方法来预测将来的指标值并/或检测异常值。 此回归过程的隐含假设是，除了周期性和趋势行为外，时序是随机确定的且随机分布。 通过周期性和趋势组件预测将来的指标值，同时忽略残差部件。 仅基于残差部件上的离群值检测来检测异常值。 如需进一步的详细信息，可参阅[时序分解章节](https://otexts.com/fpp2/decomposition.html)。

## <a name="examples"></a>示例

**每周周期性**

在下面的示例中，我们将生成一个包含每周周期性且没有趋势的序列，然后向其中添加一些离群值。 `series_decompose` 将查找并自动检测周期性，并生成与周期性组件几乎完全相同的基线。 可以在残差组件中清楚地看到我们添加的离群值。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 10.0, 15.0) - (((t%24)/10)*((t%24)/10)) // generate a series with weekly seasonality
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose(y)
| render timechart  
```

:::image type="content" source="images/samples/series-decompose1.png" alt-text="序列分解 1":::

**包含趋势的每周周期性**

在此示例中，我们向上一示例中的序列添加趋势。 首先，我们使用默认参数运行 `series_decompose`。 趋势 `avg` 默认值仅取平均值，不计算趋势。 生成的基线不包含趋势。 观察残差中的趋势时，会发现此示例明显不如上一个示例准确。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose(y)
| render timechart  
```

:::image type="content" source="images/samples/series-decompose2.png" alt-text="序列分解 1":::

接下来，我们重新运行同一示例。 由于我们预计序列中存在趋势，因此，我们在 trend 参数中指定 `linefit`。 我们可以看到，检测到了正的趋势，且基线更接近输入序列。 残差接近于零，只有离群值突出显示。可以在图表中看到序列上的所有组件。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose(y, -1, 'linefit')
| render timechart  
```

:::image type="content" source="images/samples/series-decompose3.png" alt-text="序列分解 1":::
