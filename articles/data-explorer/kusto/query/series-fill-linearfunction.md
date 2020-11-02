---
title: series_fill_linear() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_fill_linear()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 42ee58b26f419cb50faf7ca266faf2f23f66c9c7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105394"
---
# <a name="series_fill_linear"></a>series_fill_linear()

以线性方式将缺失值内插到序列中。

采用包含动态数值数组作为输入的表达式，对 missing_value_placeholder 的所有实例执行线性内插，并返回生成的数组。 如果数组的开头和结尾包含 missing_value_placeholder，则它会替换为除 missing_value_placeholder 之外的最接近值。 此功能可以关闭。 如果整个数组都由 missing_value_placeholder 组成，则数字会使用 constant_value 进行填充，如果未指定，则使用 0 进行填充。  

## <a name="syntax"></a>语法

`series_fill_linear(`x`[,` missing_value_placeholder` [,`fill_edges` [,`constant_value`]]]))`
* 将使用指定参数返回 x 的序列线性内插。
 

## <a name="arguments"></a>参数

* x：动态数组标量表达式（数值数组）。
* missing_value_placeholder：可选参数，用于指定要替换的“缺失值”的占位符。 默认值为 `double`(null)。
* fill_edges：布尔值，指示是否应将数组开头和结尾处的 missing_value_placeholder 替换为最接近值。 默认值为 True。 如果设置为 false，则会保留数组开头和结尾处的 missing_value_placeholder。
* constant_value：仅与数组相关的可选参数完全由 null 值组成。 此参数指定用于填充序列的常数值。 默认值为 0。 将此参数设置为 `double`(null) 实际上会将 null 值保留在原处。

## <a name="notes"></a>说明

* 若要在 [make-series](make-seriesoperator.md) 之后应用任何内插函数，请指定“null”作为默认值： 

    <!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
    ```kusto
    make-series num=count() default=long(null) on TimeStamp from ago(1d) to ago(1h) step 1h by Os, Browser
    ```

* missing_value_placeholder 可以是转换为实际元素类型的任何类型。 因此，无论“`double`(null)”、“`long`(null)”或“`int`(null)”都具有相同的含义。
* 如果 missing_value_placeholder 为 `double`(null)（或省略，这样含义相同），则结果可能包含 null 值。 使用其他内插函数填充这些 null 值。 目前只有 [series_outliers()](series-outliersfunction.md) 支持输入数组中有 null 值。
* 此函数会保留数组元素的原始类型。 如果 x 只包含 int 或 long 元素，则线性内插会返回舍入的内插值，而不是确切值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let data = datatable(arr: dynamic)
[
    dynamic([null, 111.0, null, 36.0, 41.0, null, null, 16.0, 61.0, 33.0, null, null]), // Array of double    
    dynamic([null, 111,   null, 36,   41,   null, null, 16,   61,   33,   null, null]), // Similar array of int
    dynamic([null, null, null, null])                                                   // Array with missing values only
];
data
| project arr, 
          without_args = series_fill_linear(arr),
          with_edges = series_fill_linear(arr, double(null), true),
          wo_edges = series_fill_linear(arr, double(null), false),
          with_const = series_fill_linear(arr, double(null), true, 3.14159)  

```

|`arr`|`without_args`|`with_edges`|`wo_edges`|`with_const`|
|---|---|---|---|---|
|[null,111.0,null,36.0,41.0,null,null,16.0,61.0,33.0,null,null]|[111.0,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,33.0,33.0]|[111.0,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,33.0,33.0]|[null,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,null,null]|[111.0,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,33.0,33.0]|
|[null,111,null,36,41,null,null,16,61,33,null,null]|[111,111,73,36,41,32,24,16,61,33,33,33]|[111,111,73,36,41,32,24,16,61,33,33,33]|[null,111,73,36,41,32,24,16,61,33,null,null]|[111,111,74,38,  41,32,24,16,61,33,33,33]|
|[null,null,null,null]|[0.0,0.0,0.0,0.0]|[0.0,0.0,0.0,0.0]|[0.0,0.0,0.0,0.0]|[3.14159,3.14159,3.14159,3.14159]|
