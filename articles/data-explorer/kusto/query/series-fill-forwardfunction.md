---
title: series_fill_forward() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_fill_forward()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 2b87a76acd3d5e9e1b38b6cab02b7563d053f510
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106181"
---
# <a name="series_fill_forward"></a>series_fill_forward()

在序列中对缺失值执行前向填充内插。

包含动态数值数组的表达式为输入。 该函数将 missing_value_placeholder 的所有实例替换为离其左侧最近的值（missing_value_placeholder 除外），并返回生成的数组。 保留 missing_value_placeholder 的最左侧实例。

## <a name="syntax"></a>语法

`series_fill_forward(`*x*`[, `*missing_value_placeholder*`])`
* 将返回序列 x，其中前向填充了 missing_value_placeholder 的所有实例。

## <a name="arguments"></a>参数

* x：动态数组标量表达式（数值数组）。 
* missing_value_placeholder：可选参数，用于指定要替换的缺失值的占位符。 默认值为“`double`(null)”。

**备注**

* 将“null”指定为默认值，以在 [make-series](make-seriesoperator.md) 之后应用内插函数： 

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
make-series num=count() default=long(null) on TimeStamp from ago(1d) to ago(1h) step 1h by Os, Browser
```

* missing_value_placeholder 可以是将要转换为实际元素类型的任何类型。 `double`(null)、`long`(null) 和 `int`(null) 具有相同的含义。
* 如果 missing_value_placeholder 为 (null) 或被省略（二者含义相同），则结果可能包含 null 值。 若要填充这些 null 值，请使用其他内插函数。 目前只有 [series_outliers()](series-outliersfunction.md) 支持在输入数组中使用 null 值。
* 这些函数会保留数组元素的原始类型。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let data = datatable(arr: dynamic)
[
    dynamic([null,null,36,41,null,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_forward = series_fill_forward(arr)  

```

|`arr`|`fill_forward`|
|---|---|
|[null,null,36,41,null,null,16,61,33,null,null]|[null,null,36,41,41,41,16,61,33,33,33]|
   
使用 [series_fill_backward](series-fill-backwardfunction.md) 或 [series-fill-const](series-fill-constfunction.md) 完成以上数组的内插。
