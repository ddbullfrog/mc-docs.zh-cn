---
title: series_fir() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 series_fir()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2019
ms.date: 09/30/2020
ms.openlocfilehash: 1765167f50368026f24d7cd069e9c01003921e47
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105518"
---
# <a name="series_fir"></a>series_fir()

对序列应用有限脉冲响应 (FIR) 滤波器。  

此函数接受一个包含动态数值数组的表达式作为输入，并应用一个有限脉冲响应滤波器。 通过指定 `filter` 系数，该函数可用于计算移动平均值、进行修匀处理和检测更改等。 该函数的输入列包含由滤波器系数组成的动态数组和静/动态数组，并且该函数会对此列应用滤波器。 它会输出新的动态数组列，其中包括滤波后的输出。  

## <a name="syntax"></a>语法

`series_fir(`*x*`,` *filter* [`,` *normalize* [`,` *center* ]]`)`

## <a name="arguments"></a>参数

* x：数值的动态数组单元。 通常是 [make-series](make-seriesoperator.md) 或 [make_list](makelist-aggfunction.md) 运算符生成的输出。
* filter：一个常量表达式，其中包含滤波器的系数（存储为由数值组成的动态数组）。
* normalize：一个可选布尔值，指示是否应将滤波器归一化。 即，将其除以系数之和。 如果滤波器包含负值，则必须将 normalize 指定为 `false`，否则结果将是 `null`。 如果未指定，则假定 normalize 采用默认值，具体取决于 filter 中是否存在负值。 如果 filter 包含至少一个负值，则假定 normalize 为 `false`。  
归一化是确保系数之和为 1 的一种简便方法。 这样做以后，滤波器不会对序列进行放大或衰减。 例如，可以通过 *filter* =[1,1,1,1] 和 *normalized* =true 指定四箱的移动平均值，这比键入 [0.25,0.25.0.25,0.25] 更加容易。
* center：一个可选布尔值，指示是对某个时段（当前时间点之前和之后）对称地应用滤波器，还是对当前时间点之前的时段应用滤波器。 默认情况下，center 为 false，这适合于流式传输数据的情况，在这种情况下，我们只能对当前的点和较旧的点应用滤波器。 但是，对于即席处理，可以将其设置为 `true`，使其与时序保持同步。 请参阅以下示例。 此参数用于控制滤波器的群延迟。

## <a name="examples"></a>示例

* 通过设置 *filter* =[1,1,1,1,1] 和 *normalize*=`true`（默认值）计算五个点的移动平均值。 请注意 *center*=`false`（默认值）与 `true` 的效果：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range t from bin(now(), 1h)-23h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS', val=dynamic([0,0,0,0,0,0,0,0,0,10,20,40,100,40,20,10,0,0,0,0,0,0,0,0]), t
| extend 5h_MovingAvg=series_fir(val, dynamic([1,1,1,1,1])),
         5h_MovingAvg_centered=series_fir(val, dynamic([1,1,1,1,1]), true, true)
| render timechart
```

此查询将返回：  
5h_MovingAvg：5 点移动平均值滤波器。 峰值经过了修匀处理，并且偏移了 (5-1)/2 = 2 小时。  
5h_MovingAvg_centered：相同，但通过设置 `center=true`，峰值会停留在其原始位置。

:::image type="content" source="images/series-firfunction/series-fir.png" alt-text="序列 fir" border="false":::

* 若要计算某个点与其前面的点之间的差异，请设置 *filter* =[1,-1]。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range t from bin(now(), 1h)-11h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS',t,value=dynamic([0,0,0,0,2,2,2,2,3,3,3,3])
| extend diff=series_fir(value, dynamic([1,-1]), false, false)
| render timechart
```

:::image type="content" source="images/series-firfunction/series-fir2.png" alt-text="序列 fir" border="false":::
