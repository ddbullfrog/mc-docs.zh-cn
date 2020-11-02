---
title: series_seasonal() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_seasonal()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 0693ab0c157111e8e94f723aa1e87941482cafcf
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105366"
---
# <a name="series_seasonal"></a>series_seasonal()

根据检测到的或给定的周期，计算序列的周期性组件。

## <a name="syntax"></a>语法

`series_seasonal(`*series* `[,` *period*`])`

## <a name="arguments"></a>参数

* series：输入数值动态数组
* period（可选）：每个周期的整数 bin 数，可能值：
    *  -1（默认值）：使用阈值为 0.7 的 [series_periods_detect()](series-periods-detectfunction.md) 自动检测期间。 如果未检测到周期性，则返回零
    * 正整数：用作周期性组件的期间
    * 任何其他值：忽略周期性，返回一系列零

## <a name="returns"></a>返回

长度与 series 输入相同的动态数组，其中包含序列的计算出的周期性组件。 周期性组件将计算为所有期间中与 bin 位置相对应的所有值的中值。

## <a name="examples"></a>示例

### <a name="auto-detect-the-period"></a>自动检测期间

在下面的示例中，将自动检测序列的期间。 检测到第一个序列的期间为六个 bin，第二个序列的周期为五个 bin。第三个序列的期间因太短而检测不到，因此会返回一系列零。 请参阅下一个有关[如何强制使用期间](#force-a-period)的示例。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print s=dynamic([2,5,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1])
| union (print s=dynamic([8,12,14,12,10,10,12,14,12,10,10,12,14,12,10,10,12,14,12,10]))
| union (print s=dynamic([1,3,5,2,4,6,1,3,5,2,4,6]))
| extend s_seasonal = series_seasonal(s)
```

|s|s_seasonal|
|---|---|
|[2,5,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1]|[1.0,2.0,3.0,4.0,3.0,2.0,1.0,2.0,3.0,4.0,3.0,2.0,1.0,2.0,3.0,4.0,3.0,2.0,1.0,2.0,3.0,4.0,3.0,2.0,1.0]|
|[8,12,14,12,10,10,12,14,12,10,10,12,14,12,10,10,12,14,12,10]|[10.0,12.0,14.0,12.0,10.0,10.0,12.0,14.0,12.0,10.0,10.0,12.0,14.0,12.0,10.0,10.0,12.0,14.0,12.0,10.0]|
|[1,3,5,2,4,6,1,3,5,2,4,6]|[0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0]|

### <a name="force-a-period"></a>强制使用期间

在此示例中，序列的期间太短，无法通过 [series_periods_detect()](series-periods-detectfunction.md) 检测到，因此我们显式地强制使用期间来获取周期模式。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print s=dynamic([1,3,5,1,3,5,2,4,6]) 
| union (print s=dynamic([1,3,5,2,4,6,1,3,5,2,4,6]))
| extend s_seasonal = series_seasonal(s,3)
```

|s|s_seasonal|
|---|---|
|[1,3,5,1,3,5,2,4,6]|[1.0,3.0,5.0,1.0,3.0,5.0,1.0,3.0,5.0]|
|[1,3,5,2,4,6,1,3,5,2,4,6]|[1.5,3.5,5.5,1.5,3.5,5.5,1.5,3.5,5.5,1.5,3.5,5.5]|
 
## <a name="next-steps"></a>后续步骤

* [series_periods_detect()](series-periods-detectfunction.md)
* [series_periods_validate()](series-periods-validatefunction.md)
