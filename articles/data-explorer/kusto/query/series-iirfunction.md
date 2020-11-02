---
title: series_iir() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_iir()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/20/2019
ms.date: 09/30/2020
ms.openlocfilehash: f796fcef891072408d9c8a2fff6cce6534fcc7a5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105384"
---
# <a name="series_iir"></a>series_iir()

对序列应用无限脉冲响应滤波器。  

此函数接受一个包含动态数值数组的表达式作为输入，并应用一个无限脉冲响应滤波器。 通过指定滤波器系数，可以将此函数用于：
* 计算序列的累计和
* 应用平滑操作
* 应用各种高通、带通和低通滤波器

此函数的输入列包含由滤波器的 a 系数和 b 系数组成的动态数组和两个静/动态数组，并且此函数会对该列应用滤波器。 它会输出新的动态数组列，其中包括滤波后的输出。  

## <a name="syntax"></a>语法

`series_iir(`*x*`,` *b* `,` *a*`)`

## <a name="arguments"></a>参数

* x：动态数组单元格（数值数组），通常是 [make-series](make-seriesoperator.md) 或 [make_list](makelist-aggfunction.md) 运算符生成的输出。
* b：一个常量表达式，其中包含滤波器的分子系数（存储为由数值组成的动态数组）。
* a：一个常量表达式，例如 b。 包含滤波器的分母系数。

> [!IMPORTANT]
> `a` 的第一个元素（即 `a[0]`）不得为零，以避免除以 0 的现象发生。 请查看[下面的公式](#the-filters-recursive-formula)。

## <a name="the-filters-recursive-formula"></a>滤波器的递归公式

* 假设输入数组为 X，系数数组 a 和 b 的长度分别为 n_a 和 n_b。 滤波器的传递函数（会生成输出数组 Y）的定义如下：

<div align="center">
Y<sub>i</sub> = a<sub>0</sub><sup>-1</sup>(b<sub>0</sub>X<sub>i</sub>
 + b<sub>1</sub>X<sub>i-1</sub> + ... + b<sub>n<sub>b</sub>-1</sub>X<sub>i-n<sub>b</sub>-1</sub>
 - a<sub>1</sub>Y<sub>i-1</sub>-a<sub>2</sub>Y<sub>i-2</sub> - ... - a<sub>n<sub>a</sub>-1</sub>Y<sub>i-n<sub>a</sub>-1</sub>)
</div>

## <a name="example"></a>示例

计算累计和。 使用系数 a=[1,-1] 且 b=[1] 的 iir 滤波器：  

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let x = range(1.0, 10, 1);
print x=x, y = series_iir(x, dynamic([1]), dynamic([1,-1]))
| mv-expand x, y
```

| x | y |
|:--|:--|
|1.0|1.0|
|2.0|3.0|
|3.0|6.0|
|4.0|10.0|

将其包装在函数中的方法如下：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let vector_sum=(x:dynamic)
{
  let y=array_length(x) - 1;
  toreal(series_iir(x, dynamic([1]), dynamic([1, -1]))[y])
};
print d=dynamic([0, 1, 2, 3, 4])
| extend dd=vector_sum(d)
```

|d            |dd  |
|-------------|----|
|`[0,1,2,3,4]`|`10`|
