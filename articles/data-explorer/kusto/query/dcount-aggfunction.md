---
title: dcount()（聚合函数） - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 dcount()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 7ad9ed53ebcc10a70e2df043bcdb07161ecf967e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104281"
---
# <a name="dcount-aggregation-function"></a>dcount()（聚合函数）

返回摘要组中标量表达式占用的非重复值数的估计值。

> [!NOTE]
> `dcount()` 聚合函数主要用于估算大型集的基数。 它以性能确定准确性，并可能在执行间返回不同的结果。 输入的顺序可能会影响其输出。

## <a name="syntax"></a>语法

... `|` `summarize` `dcount` `(`*`Expr`* [, *`Accuracy`* ]`)` ...

## <a name="arguments"></a>参数

* Expr：要对其非重复值进行计数的标量表达式。
* *准确度* ：一个可选的 `int` 文本，用于定义请求的估计准确度。 有关支持的值，请参阅下文。 如果未指定，则使用默认值 `1`。

## <a name="returns"></a>返回

返回对组中 `Expr` 的非重复值数的估计值。

## <a name="example"></a>示例

```kusto
PageViewLog | summarize countries=dcount(country) by continent
```

:::image type="content" source="images/dcount-aggfunction/dcount.png" alt-text="D 计数":::

获取按 `G` 分组的 `V` 的非重复值的精确计数。

```kusto
T | summarize by V, G | summarize count() by G
```

由于 `V` 的非重复值与 `G` 的非重复值的数量相乘，因此此计算需要大量内部内存。
这可能会导致内存错误或执行时间过长。 
`dcount()` 提供了快速可靠的替代方法：

```kusto
T | summarize dcount(B) by G | count
```

## <a name="estimation-accuracy"></a>估计准确度

`dcount()` 聚合函数使用 HyperLogLog (HLL) 算法的变体，该算法对集基数进行随机估算。 该算法提供一个“旋钮”，可用于平衡每个内存大小的准确度和执行时间：

|精确度|错误 (%)|条目数   |
|--------|---------|--------------|
|       0|      1.6|2<sup>12</sup>|
|       1|      0.8|2<sup>14</sup>|
|       2|      0.4|2<sup>16</sup>|
|       3|     0.28|2<sup>17</sup>|
|       4|      0.2|2<sup>18</sup>|

> [!NOTE]
> “条目数”列是 HLL 实现中 1 字节计数器的数目。

如果集基数足够小，则该算法包括以下有关执行理想计数（零错误）的规定：
* 当准确度等级为 `1` 时，将返回 1000 个值
* 当准确度等级为 `2` 时，将返回 8000 个值

错误边界基于概率，而不是基于理论界限。 值是错误分布的标准偏差 (sigma)，99.7% 的估计值的相对误差小于 3 x sigma。

下图显示所有受支持的准确度设置的相对估计误差的概率分布函数，以百分比为单位：

:::image type="content" border="false" source="images/dcount-aggfunction/hll-error-distribution.png" alt-text="D 计数":::
