---
title: series_periods_validate() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_periods_validate()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2019
ms.date: 09/30/2020
ms.openlocfilehash: 2032f7649b1f81731db66455e39552bcd4ca0d69
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105370"
---
# <a name="series_periods_validate"></a>series_periods_validate()

检查时序是否包含给定长度的定期模式。  

通常，测量应用程序流量的指标以每周或每天的时间段为特征。 可通过运行 `series_periods_validate()`（检查每周和每天时间段）来确认此时间段。

该函数采用一个列作为输入，该列中包含动态时序数组（通常是 [make-series](make-seriesoperator.md) 运算符的结果输出）以及定义待验证时间段长度的一个或多个 `real` 数字。

该函数输出两个列：
* *periods* ：一个动态数组，包含要验证的时间段（在输入中提供）。
* *scores* ：一个动态数组，包含一个界于 0 到 1 之间的分数。 该分数显示时间段在 periods 数组中其各自位置上的重要性。

## <a name="syntax"></a>语法

`series_periods_validate(`*x*`,` *period1* [ `,` *period2* `,` . . . ] `)`

## <a name="arguments"></a>参数

* x：动态数组标量表达式，它是一个数值数组，通常是 [make-series](make-seriesoperator.md) 或 [make_list](makelist-aggfunction.md) 运算符生成的结果输出。
* period1、period2 等：指定要验证的时间段的 `real` 数字，以箱大小为单位 。 例如，如果将序列按 1 小时的箱分箱，则一个每周时间段为 168 个箱。

> [!IMPORTANT]
> * 对于每个 period 参数，其最小值是 4，最大值是输入序列长度的一半。 对于在这些界限之外的 period 参数，输出分数将会是 0。
>
> * 输入时序必须有规律（也就是必须在常数箱中聚合），并且如果是使用 [make-series](make-seriesoperator.md) 创建的，则输入时序总是有规律。 否则，输出就没有意义。
> 
> * 该函数最多可接受 16 个要验证的时间段。

## <a name="example"></a>示例

下面的查询嵌入某应用程序流量一个月的快照，每天聚合两次（箱大小为 12 小时）。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| render linechart 
```

:::image type="content" source="images/series-periods/series-periods.png" alt-text="序列时间段":::

如果对此序列运行 `series_periods_validate()` 来验证一个每周时间段（长度为 14 个点），它会产生一个高分，而在验证一个 5 天时间段（长度为 10 个点）时分数为 0。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| project series_periods_validate(y, 14.0, 10.0)
```

| series\_periods\_validate\_y\_periods  | series\_periods\_validate\_y\_scores |
|-------------|-------------------|
| [14.0, 10.0] | [0.84,0.0]  |
