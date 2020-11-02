---
title: render 运算符 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 render 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/29/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: cd5eb77db1251f3446e840ffe0251827ee902720
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105024"
---
# <a name="render-operator"></a>render 运算符

指示用户代理以特定方式呈现查询结果。

```kusto
range x from 0.0 to 2*pi() step 0.01 | extend y=sin(x) | render linechart
```

> [!NOTE]
> * Render 运算符应当是查询中的最后一个运算符，仅与生成单个表格数据流结果的查询一起使用。
> * Render 运算符不修改数据。 它在结果的扩展属性中注入注释（“可视化效果”）。 注释包含由查询中的运算符提供的信息。
> * 可视化效果信息的解释由用户代理来完成。 不同的代理（例如 Kusto.Explorer、Kusto.WebExplorer）可能支持不同的可视化效果。

## <a name="syntax"></a>语法

*T* `|` `render` *Visualization* [`with` `(` *PropertyName* `=` *PropertyValue* [`,` ...] `)`]

其中：

* Visualization 指示要使用的可视化效果的种类。 支持的值包括：

::: zone pivot="azuredataexplorer"

|*可视化*     |描述|
|--------------------|-|
| `anomalychart`     | 与 timechart 类似，但使用 [series_decompose_anomalies](./series-decompose-anomaliesfunction.md) 函数[突出显示异常](./samples.md#get-more-out-of-your-data-in-kusto-with-machine-learning)。 |
| `areachart`        | 面积图。 第一列是 x 轴，应当为数值列。 其他数值列是 y 轴。 |
| `barchart`         | 第一列是 x 轴，可以是文本、日期/时间或数值。 其他列是数值，显示为水平条带。|
| `card`             | 第一个结果记录被视为标量值集，并显示为卡片。 |
| `columnchart`      | 像 `barchart` 一样，但带有竖直条带而不是水平条带。|
| `ladderchart`      | 最后两列是 x 轴，其他列是 y 轴。|
| `linechart`        | 折线图。 第一列是 x 轴，应当为数值列。 其他数值列是 y 轴。 |
| `piechart`         | 第一列是颜色轴，第二列是数值。 |
| `pivotchart`       | 显示数据透视表和图表。 用户可以通过交互方式选择数据、列、行和各种图表类型。 |
| `scatterchart`     | 点图。 第一列是 x 轴，应当为数值列。 其他数值列是 y 轴。 |
| `stackedareachart` | 堆积面积图。 第一列是 x 轴，应当为数值列。 其他数值列是 y 轴。 |
| `table`            | 默认设置 - 结果显示为表。|
| `timechart`        | 折线图。 第一列是 x 轴，且应为日期/时间。 其他（数值）列是 y 轴。 有一个字符串列，其值用来对数值列进行“分组”并在图表中创建不同的线条（将忽略其他字符串列）。 |
| `timepivot`        | 在事件时间线上进行交互式导航（沿时间轴旋转）|

::: zone-end

::: zone pivot="azuremonitor"

|*可视化*     |描述|
|--------------------|-|
| `areachart`        | 面积图。 第一列是 x 轴，应当为数值列。 其他数值列是 y 轴。 |
| `barchart`         | 第一列是 x 轴，可以是文本、日期/时间或数值。 其他列是数值，显示为水平条带。|
| `columnchart`      | 像 `barchart` 一样，但带有竖直条带而不是水平条带。|
| `piechart`         | 第一列是颜色轴，第二列是数值。 |
| `scatterchart`     | 点图。 第一列是 x 轴，应当为数值列。 其他数值列是 y 轴。 |
| `table`            | 默认设置 - 结果显示为表。|
| `timechart`        | 折线图。 第一列是 x 轴，且应为日期/时间。 其他（数值）列是 y 轴。 有一个字符串列，其值用来对数值列进行“分组”并在图表中创建不同的线条（将忽略其他字符串列）。|

::: zone-end

* PropertyName/PropertyValue 指示在呈现时要使用的其他信息。
  所有属性都是可选的。 支持的属性如下：

::: zone pivot="azuredataexplorer"

|PropertyName|PropertyValue                                                                   |
|--------------|----------------------------------------------------------------------------------|
|`accumulate`  |是否将每个度量的值加到其所有前导度量中。 （`true` 或 `false`）|
|`kind`        |可视化效果种类的进一步细化。 请参阅下文。                         |
|`legend`      |是否显示图例（`visible` 或 `hidden`）。                       |
|`series`      |以逗号分隔的列列表，其中的每记录值组合定义了记录所属的系列。|
|`ymin`        |要在 Y 轴上显示的最小值。                                      |
|`ymax`        |要在 Y 轴上显示的最大值。                                      |
|`title`       |可视化效果的标题（`string` 类型）。                                |
|`xaxis`       |如何缩放 x 轴（`linear` 或 `log`）。                                      |
|`xcolumn`     |结果中的哪一列用于 x 轴。                                |
|`xtitle`      |x 轴的标题（`string` 类型）。                                       |
|`yaxis`       |如何缩放 y 轴（`linear` 或 `log`）。                                      |
|`ycolumns`    |由逗号分隔的列列表，其中包含根据 x 列的值提供的值。|
|`ysplit`      |如何拆分多个可视化效果。 请参阅下文。                               |
|`ytitle`      |y 轴的标题（`string` 类型）。                                       |
|`anomalycolumns`|仅限与 `anomalychart` 相关的属性。 将被视为异常系列并在图表中显示为点的列的逗号分隔列表|

::: zone-end

::: zone pivot="azuremonitor"

|PropertyName|PropertyValue                                                                   |
|--------------|----------------------------------------------------------------------------------|
|`kind`        |可视化效果种类的进一步细化。 请参阅下文。                         |
|`series`      |以逗号分隔的列列表，其中的每记录值组合定义了记录所属的系列。|
|`title`       |可视化效果的标题（`string` 类型）。                                |
|`yaxis`       |如何缩放 y 轴（`linear` 或 `log`）。                                      |

::: zone-end

可以通过提供 `kind` 属性来进一步详细阐述某些可视化效果。
这些是：

|*可视化*|`kind`             |描述                        |
|---------------|-------------------|-----------------------------------|
|`areachart`    |`default`          |每个“区域”都独立存在。     |
|               |`unstacked`        |与 `default` 相同。                 |
|               |`stacked`          |将“区域”堆叠到右侧。        |
|               |`stacked100`       |将“区域”堆叠到右侧，并将每个区域拉伸到与其他区域相同的宽度。|
|`barchart`     |`default`          |每个“条形”都独立存在。      |
|               |`unstacked`        |与 `default` 相同。                 |
|               |`stacked`          |对“条形”进行堆叠。                      |
|               |`stacked100`       |对“条形”进行堆叠，并将每个条形拉伸到与其他条形相同的宽度。|
|`columnchart`  |`default`          |每个“柱形”都独立存在。   |
|               |`unstacked`        |与 `default` 相同。                 |
|               |`stacked`          |将“柱形”一个接一个地堆叠。|
|               |`stacked100`       |对“柱形”进行堆叠，并将每个柱形拉伸到与其他柱形相同的高度。|
|`scatterchart` |`map`              |预期的柱形为 [经度，纬度] 或 GeoJSON 点。 系列列是可选的。|
|`piechart`     |`map`              |预期的柱形为 [经度，纬度] 或 GeoJSON 点、颜色轴和数值。 在 Kusto 资源管理器桌面中受支持。|

::: zone pivot="azuredataexplorer"

某些可视化效果支持拆分为多个 y 轴值：

|`ysplit`  |描述                                                       |
|----------|------------------------------------------------------------------|
|`none`    |为所有系列数据显示单个 y 轴。 （默认值）       |
|`axes`    |单个图表将显示多个 y 轴（每个系列一个）。|
|`panels`  |为每个 `ycolumn` 值呈现一个图表（直至达到某个限制）。|

::: zone-end

> [!NOTE]
> render 运算符的数据模型将表格数据视为具有下述三种列的数据：
>
> * X 轴列（由 `xcolumn` 属性指示）。
> * 系列列（由 `series` 属性指示的任意数目的列。）对于每个记录，这些列的组合值定义单个系列，而图表的系列数量与非重复组合值的数量相同。
> * Y 轴列（由 `ycolumns` 属性指示的任意数目的列）。
  对于每个记录，该系列都具有与 y 轴列一样多的度量（图表中的“点”）。

> [!TIP]
> 
> * 可使用 `where`、`summarize` 和 `top` 来限制显示的数据量。
> * 对数据进行排序以定义 x 轴的顺序。
> * 用户代理可以自由地“推测”查询未指定的属性的值。 需要特别注意的是，在结果架构中提高“不感兴趣的”列可能会导致推测错误。 如果出现这种情况，请尝试通过 project-away 运算符排除此类列。 

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
range x from -2 to 2 step 0.1
| extend sin = sin(x), cos = cos(x)
| extend x_sign = iif(x > 0, "x_pos", "x_neg")
| extend sum_sign = iif(sin + cos > 0, "sum_pos", "sum_neg")
| render linechart with  (ycolumns = sin, cos, series = x_sign, sum_sign)
```

::: zone pivot="azuredataexplorer"

[教程中的呈现示例](./tutorial.md#render-display-a-chart-or-table)。

[异常检测](./samples.md#get-more-out-of-your-data-in-kusto-with-machine-learning)

::: zone-end
