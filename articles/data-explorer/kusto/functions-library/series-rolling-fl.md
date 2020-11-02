---
title: series_rolling_fl() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中用户定义的函数 series_rolling_fl()。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/08/2020
ms.date: 10/29/2020
ms.openlocfilehash: f1fa4cfcbe82b2d989eaf46172d251d9c83e59e6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105115"
---
# <a name="series_rolling_fl"></a>series_rolling_fl()


函数 `series_rolling_fl()` 对序列应用滚动聚合。 它采用包含多个序列（动态数值数组）的表，并对每个序列应用滚动聚合函数。

> [!NOTE]
> * `series_rolling_fl()` 是 [UDF（用户定义的函数）](../query/functions/user-defined-functions.md)。
> * 此函数包含内联 Python，需要在群集上[启用 python() 插件](../query/pythonplugin.md#enable-the-plugin)。 有关详细信息，请参阅[用法](#usage)。

## <a name="syntax"></a>语法

`T | invoke series_rolling_fl(`*y_series*`,` *y_rolling_series*`,` *n*`,` *aggr*`,` *aggr_params*`,` *center*`)`

## <a name="arguments"></a>参数

* y_series：要拟合的序列所在的输入表列的名称。
* y_rolling_series：存储滚动聚合序列的列的名称。
* n：滚动窗口的宽度。
* aggr：要使用的聚合函数的名称。 请参阅[聚合函数](#aggregation-functions)。
* aggr_params：聚合函数的可选参数。
* center：一个可选布尔值，指示滚动窗口是否为以下选项之一：
    * 在当前点前后对称地应用，或者 
    * 从当前点向后应用。 <br>
    默认情况下，对于流数据的计算，center 为 false。

## <a name="aggregation-functions"></a>聚合函数

此函数支持从序列计算标量的 [numpy](https://numpy.org/) 或 [scipy.stats](https://docs.scipy.org/doc/scipy/reference/stats.html#module-scipy.stats) 中的任何聚合函数。 以下列表并不详尽：

* [`sum`](https://numpy.org/doc/stable/reference/generated/numpy.sum.html#numpy.sum) 
* [`mean`](https://numpy.org/doc/stable/reference/generated/numpy.mean.html?highlight=mean#numpy.mean)
* [`min`](https://numpy.org/doc/stable/reference/generated/numpy.amin.html#numpy.amin)
* [`max`](https://numpy.org/doc/stable/reference/generated/numpy.amax.html)
* [`ptp (max-min)`](https://numpy.org/doc/stable/reference/generated/numpy.ptp.html)
* [`percentile`](https://numpy.org/doc/stable/reference/generated/numpy.percentile.html)
* [`median`](https://numpy.org/doc/stable/reference/generated/numpy.median.html)
* [`std`](https://numpy.org/doc/stable/reference/generated/numpy.std.html)
* [`var`](https://numpy.org/doc/stable/reference/generated/numpy.var.html)
* [`gmean`（几何平均值）](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.gmean.html)
* [`hmean`（调和平均值）](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.hmean.html)
* [`mode`（最常见的值）](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.mode.html)
* [`moment`（第 n<sup></sup> 个时刻）](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.moment.html)
* [`tmean`（剪裁平均值）](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.tmean.html)
* [`tmin`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.tmin.html) 
* [`tmax`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.tmax.html)
* [`tstd`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.tstd.html)
* [`iqr`（四分位差）](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.iqr.html) 

## <a name="usage"></a>使用情况

`series_rolling_fl()` 是用户定义的[表格函数](../query/functions/user-defined-functions.md#tabular-function)，需使用 [invoke 运算符](../query/invokeoperator.md)进行应用。 可以在查询中嵌入其代码，或将其安装在数据库中。 用法选项有两种：临时使用和永久使用。 请参阅下面选项卡上的示例。

# <a name="ad-hoc"></a>[临时](#tab/adhoc)

如果是临时使用，请使用 [let 语句](../query/letstatement.md)嵌入该函数的代码。 不需要权限。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let series_rolling_fl = (tbl:(*), y_series:string, y_rolling_series:string, n:int, aggr:string, aggr_params:dynamic=dynamic([null]), center:bool=true)
{
    let kwargs = pack('y_series', y_series, 'y_rolling_series', y_rolling_series, 'n', n, 'aggr', aggr, 'aggr_params', aggr_params, 'center', center);
    let code =
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_rolling_series = kargs["y_rolling_series"]\n'
        'n = kargs["n"]\n'
        'aggr = kargs["aggr"]\n'
        'aggr_params = kargs["aggr_params"]\n'
        'center = kargs["center"]\n'
        'result = df\n'
        'in_s = df[y_series]\n'
        'func = getattr(np, aggr, None)\n'
        'if not func:\n'
        '    import scipy.stats\n'
        '    func = getattr(scipy.stats, aggr)\n'
        'if func:\n'
        '    result[y_rolling_series] = list(pd.Series(in_s[i]).rolling(n, center=center, min_periods=1).apply(func, args=aggr_params).values for i in range(len(in_s)))\n'
        '\n';
    tbl
    | evaluate python(typeof(*), code, kwargs)
}
;
//
//  Calculate rolling median of 9 elements
//
demo_make_series1
| make-series num=count() on TimeStamp step 1h by OsVer
| extend rolling_med = dynamic(null)
| invoke series_rolling_fl('num', 'rolling_med', 9, 'median')
| render timechart
```

# <a name="persistent"></a>[Persistent](#tab/persistent)

如果是永久使用，请使用 [.create 函数](../management/create-function.md)。 创建函数需要有[数据库用户权限](../management/access-control/role-based-authorization.md)。

### <a name="one-time-installation"></a>一次性安装

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
.create-or-alter function with (folder = "Packages\\Series", docstring = "Rolling window functions on a series")
series_rolling_fl(tbl:(*), y_series:string, y_rolling_series:string, n:int, aggr:string, aggr_params:dynamic, center:bool=true)
{
    let kwargs = pack('y_series', y_series, 'y_rolling_series', y_rolling_series, 'n', n, 'aggr', aggr, 'aggr_params', aggr_params, 'center', center);
    let code =
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_rolling_series = kargs["y_rolling_series"]\n'
        'n = kargs["n"]\n'
        'aggr = kargs["aggr"]\n'
        'aggr_params = kargs["aggr_params"]\n'
        'center = kargs["center"]\n'
        'result = df\n'
        'in_s = df[y_series]\n'
        'func = getattr(np, aggr, None)\n'
        'if not func:\n'
        '    import scipy.stats\n'
        '    func = getattr(scipy.stats, aggr)\n'
        'if func:\n'
        '    result[y_rolling_series] = list(pd.Series(in_s[i]).rolling(n, center=center, min_periods=1).apply(func, args=aggr_params).values for i in range(len(in_s)))\n'
        '\n';
    tbl
    | evaluate python(typeof(*), code, kwargs)
}
```

### <a name="usage"></a>使用情况

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
//
//  Calculate rolling median of 9 elements
//
demo_make_series1
| make-series num=count() on TimeStamp step 1h by OsVer
| extend rolling_med = dynamic(null)
| invoke series_rolling_fl('num', 'rolling_med', 9, 'median', dynamic([null]))
| render timechart
```

---

:::image type="content" source="images/series-rolling-fl/rolling-median-9.png" alt-text="描述了 9 个元素的滚动中值的图形" border="false":::

## <a name="additional-examples"></a>其他示例

以下示例假定已安装该函数：

1. 计算 15 个元素的滚动最小值、最大值和第 75 个百分位
    
    <!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
    ```kusto
    //
    //  Calculate rolling min, max & 75th percentile of 15 elements
    //
    demo_make_series1
    | make-series num=count() on TimeStamp step 1h by OsVer
    | extend rolling_min = dynamic(null), rolling_max = dynamic(null), rolling_pct = dynamic(null)
    | invoke series_rolling_fl('num', 'rolling_min', 15, 'min', dynamic([null]))
    | invoke series_rolling_fl('num', 'rolling_max', 15, 'max', dynamic([null]))
    | invoke series_rolling_fl('num', 'rolling_pct', 15, 'percentile', dynamic([75]))
    | render timechart
    ```
    
    :::image type="content" source="images/series-rolling-fl/graph-rolling-15.png" alt-text="描述了 9 个元素的滚动中值的图形" border="false":::

1. 计算滚动剪裁平均值
        
    <!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
    ```kusto
    //
    //  Calculate rolling trimmed mean
    //
    range x from 1 to 100 step 1
    | extend y=iff(x % 13 == 0, 2.0, iff(x % 23 == 0, -2.0, rand()))
    | summarize x=make_list(x), y=make_list(y)
    | extend yr = dynamic(null)
    | invoke series_rolling_fl('y', 'yr', 7, 'tmean', pack_array(pack_array(-2, 2), pack_array(false, false))) //  trimmed mean: ignoring values outside [-2,2] inclusive
    | render linechart
    ```
    
    :::image type="content" source="images/series-rolling-fl/rolling-trimmed-mean.png" alt-text="描述了 9 个元素的滚动中值的图形" border="false":::
    