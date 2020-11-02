---
title: series_pearson_correlation() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_pearson_correlation()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/31/2019
ms.date: 09/30/2020
ms.openlocfilehash: 9e439bf9f682005ccb79b110ec594fadf052a5e4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105367"
---
# <a name="series_pearson_correlation"></a>series_pearson_correlation()

计算两个数值序列输入的皮尔逊相关系数。

## <a name="syntax"></a>语法

`series_pearson_correlation(`*Series1*`,` *Series2*`)`

## <a name="arguments"></a>参数

* Series1、Series2：用于计算相关系数的输入数值数组。 所有参数都必须是长度相同的动态数组。 

## <a name="returns"></a>返回

两个输入之间计算的皮尔逊相关系数。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 结果。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range s1 from 1 to 5 step 1 | extend s2 = 2*s1 // Perfect correlation
| summarize s1 = make_list(s1), s2 = make_list(s2)
| extend correlation_coefficient = series_pearson_correlation(s1,s2)
```

|s1|s2|correlation_coefficient|
|---|---|---|
|[1,2,3,4,5]|[2,4,6,8,10]|1|
