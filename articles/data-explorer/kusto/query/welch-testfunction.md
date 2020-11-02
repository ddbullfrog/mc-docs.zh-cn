---
title: welch_test() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 welch_test()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: d84f093b346937502e36b3d05d43eb9a5e2ec1ae
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104372"
---
# <a name="welch_test"></a>welch_test()

计算 Welch-test 函数的 p_value

```kusto
// s1, s2 values are from https://en.wikipedia.org/wiki/Welch%27s_t-test
print
    s1 = dynamic([27.5, 21.0, 19.0, 23.6, 17.0, 17.9, 16.9, 20.1, 21.9, 22.6, 23.1, 19.6, 19.0, 21.7, 21.4]),
    s2 = dynamic([27.1, 22.0, 20.8, 23.4, 23.4, 23.5, 25.8, 22.0, 24.8, 20.2, 21.9, 22.1, 22.9, 20.5, 24.4])
| mv-expand s1 to typeof(double), s2 to typeof(double)
| summarize m1=avg(s1), v1=variance(s1), c1=count(), m2=avg(s2), v2=variance(s2), c2=count()
| extend pValue=welch_test(m1,v1,c1,m2,v2,c2)

// pValue = 0.021
```

## <a name="syntax"></a>语法

`welch_test(`*mean1*`, `*variance1*`, `*count1*`, `*mean2*`, `*variance2*`, `*count2*`)`

## <a name="arguments"></a>参数

* mean1：表示第一个序列的平均值的表达式
* variance1：表示第一个序列的方差值的表达式
* count1：表示第一个序列中的值计数的表达式
* mean2：表示第二个序列的平均值的表达式
* variance2：表示第二个序列的方差值的表达式
* count2：表示第二个序列中的值计数的表达式

## <a name="returns"></a>返回

在统计学上，Welch t 检验是一种双样本位置检验，用于检验两个总体均值相等的假设。 Welch t 检验是对学生 t 检验的一种改编，当两个样本的方差和样本大小不相等时，其可靠性更高。 这些检验通常称为“未配对”或“独立样本”t 检验。 当比较的两个样本的统计单位不重叠时，通常会应用这些检验。 Welch t 检验不如学生 t 检验常用，读者可能不太熟悉。 该检验也称为“Welch 不等方差 t 检验”或“不等方差 t 检验”。
