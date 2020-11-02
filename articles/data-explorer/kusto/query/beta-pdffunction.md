---
title: beta_pdf() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 beta_pdf()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 387b8cee2a6f77ceb47c5adf2949467cb62f67b9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103738"
---
# <a name="beta_pdf"></a>beta_pdf()

返回概率密度 beta 函数。

```kusto
beta_pdf(0.2, 10.0, 50.0)
```

Beta 分布通常用于研究样本中某个因素的变化情况（用百分数表示），如一天中人们看电视所花时间的比例。

## <a name="syntax"></a>语法

`beta_pdf(`*x*`, `*alpha*`, `*beta*`)`

## <a name="arguments"></a>参数

* x：一个值，用于对函数进行计算。
* alpha：分布的一个参数。
* beta：分布的一个参数。

## <a name="returns"></a>返回

* 概率 beta 密度函数。

**说明**

如果任何参数都不是数字，则 beta_pdf() 会返回 null 值。

如果 x ≤ 0 或 1 ≤ x，beta_pdf() 会返回 NAN 值。

如果 alpha ≤ 0 或 beta ≤ 0，则 beta_pdf() 会返回 NAN 值。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(x:double, alpha:double, beta:double, comment:string)
[
    0.5, 10.0, 20.0, "Valid input",
    1.5, 10.0, 20.0, "x > 1, yields NaN",
    double(-10), 10.0, 20.0, "x < 0, yields NaN",
    0.1, double(-1.0), 20.0, "alpha is < 0, yields NaN"
]
| extend r = beta_pdf(x, alpha, beta)
```

|x|alpha|beta|comment|r|
|---|---|---|---|---|
|0.5|10 个|20 个|有效输入|0.746176019310951|
|1.5|10 个|20 个|x > 1，生成 NAN|NaN|
|-10|10 个|20 个|x < 0，生成 NAN|NaN|
|0.1|-1|20 个|alpha < 0，生成 NAN|NaN|

**参考**

* 若要计算 beta 累积概率密度函数的反函数，请参阅 [beta-inv()](./beta-invfunction.md)。
* 有关标准累积 beta 分布函数，请参阅 [beta-cdf()](./beta-cdffunction.md)。
