---
title: series_periods_detect() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 series_periods_detect()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2019
ms.date: 09/30/2020
ms.openlocfilehash: 474fa6a8fadf842176090e52121e6fc43154f7a7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105369"
---
# <a name="series_periods_detect"></a>series_periods_detect()

找出一个时序中最重要的周期。  

通常，可以按两个重要的期间将测量应用程序流量的指标特征化：每周和每天。 函数 `series_periods_detect()` 检测时序中的这两个主导期间。  
此函数采用以下输入：
* 一个包含时序动态数组的列。 通常，该列是 [make-series](make-seriesoperator.md) 运算符生成的输出。
* 两个 `real` 数字，定义最小和最大期间大小（要搜索的箱数）。 例如，对于 1 小时箱，每日周期的大小将为 24。 
* 一个 `long` 数字，定义函数要搜索的总期间数。 

该函数输出两个列：
* periods：一个动态数组，其中包含已找到的期间（以箱大小为单位，按其分数排序）。
* scores：一个动态数组，其中包含 0 到 1 之间的值。 每个数组都会度量 periods 数组中的一个期间在其相应位置上的重要性。
 
## <a name="syntax"></a>语法

`series_periods_detect(`*x*`,` *min_period*`,` *max_period*`,` *num_periods*`)`

## <a name="arguments"></a>参数

* x：动态数组标量表达式（数值数组），通常是 [make-series](make-seriesoperator.md) 或 [make_list](makelist-aggfunction.md) 运算符生成的输出。
* min_period：一个 `real` 数字，指定要搜索的最小周期。
* max_period：一个 `real` 数字，指定要搜索的最大周期。
* num_periods：一个 `long` 数字，指定所需的最大周期数。 此数字将是输出动态数组的长度。

> [!IMPORTANT]
> * 该算法可检测至少包含 4 个点且最多包含序列长度一半的期间。 
>
> * 请将 min_period 设置为略低于预计在时序中找到的期间数，而将 max_period 设置为略高于该期间数。 例如，如果你有每小时聚合的信号，并且你要查找每日和每周期间（分别是 24 小时和 168 小时），则可以设置 *min_period* =0.8\*24， *max_period* =1.2\*168，围绕这些期间留出 20% 的富余。
>
> * 输入时序必须有规律。 也就是说，聚合在常量箱中（使用 [make-series](make-seriesoperator.md) 创建的时序始终满足此条件）。 否则，输出就没有意义。

## <a name="example"></a>示例

下面的查询嵌入了应用程序某个月流量的快照，一天聚合两次。 箱大小为 12 小时。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| render linechart 
```

:::image type="content" source="images/series-periods/series-periods.png" alt-text="序列期间":::

对此序列运行 `series_periods_detect()` 将得到每周周期（长度为 14 个点）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| project series_periods_detect(y, 0.0, 50.0, 2)
```

| series\_periods\_detect\_y\_periods  | series\_periods\_detect\_y\_periods\_scores |
|-------------|-------------------|
| [14.0, 0.0] | [0.84, 0.0]  |


> [!NOTE] 
> 由于采样太粗糙（箱大小为 12 小时），因此没有找到也可以在图表中看到的每日期间，因此包含 2 箱的每日期间低于算法所要求的包含 4 个点的最小期间大小。
