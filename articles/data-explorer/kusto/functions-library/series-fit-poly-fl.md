---
title: series_fit_poly_fl() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中用户定义的函数 series_fit_poly_fl()。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/08/2020
ms.date: 09/24/2020
ms.openlocfilehash: be030dc03157f70caf2d1d0ac2b629364e27f951
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146874"
---
# <a name="series_fit_poly_fl"></a>series_fit_poly_fl()

`series_fit_poly_fl()` 函数对序列应用多项式回归。 它采用包含多个序列（动态数值数组）的表，并使用[多项式回归](https://en.wikipedia.org/wiki/Polynomial_regression)为每个序列生成拟合效果最佳的高阶多项式。 此函数针对序列范围返回多项式系数和内插多项式。

> [!NOTE]
> `series_fit_poly_fl()` 是 [UDF（用户定义的函数）](../query/functions/user-defined-functions.md)。 此函数包含内联 Python，需要在群集上[启用 python() 插件](../query/pythonplugin.md#enable-the-plugin)。 有关详细信息，请参阅[用法](#usage)。 对于间距均匀的序列（由 [make-series 运算符](../query/make-seriesoperator.md)创建）的线性回归，请使用本机函数 [series_fit_line()](../query/series-fit-linefunction.md)。

## <a name="syntax"></a>语法

`T | invoke series_fit_poly_fl(`*y_series*`,` *y_fit_series*`,` *fit_coeff*`,` *degree*`, [`*x_series*`,` *x_istime*]`)`
  
## <a name="arguments"></a>参数

* y_series：包含[依赖变量](https://en.wikipedia.org/wiki/Dependent_and_independent_variables)的输入表列的名称。 即，要拟合的序列。
* y_fit_series：存储最佳拟合序列的列的名称。
* fit_coeff：存储最佳拟合多项式系数的列的名称。
* degree：要拟合的多项式所需的阶。 例如，1 用于线性回归，2 用于二次回归，等等。
* x_series：包含[独立变量](https://en.wikipedia.org/wiki/Dependent_and_independent_variables)的列的名称，即 x 轴（或时间轴）。 此参数为可选，只有[间距不均匀的序列](https://en.wikipedia.org/wiki/Unevenly_spaced_time_series)才需要。 默认值为空字符串，因为对于间距均匀的序列的回归，x 是冗余的。
* x_istime：此布尔参数为可选。 仅当指定了 x_series 并且它是 datetime 的向量时，才需要此参数。

## <a name="usage"></a>使用情况

`series_fit_poly_fl()` 是用户定义的[表格函数](../query/functions/user-defined-functions.md#tabular-function)，需使用 [invoke 运算符](../query/invokeoperator.md)进行应用。 可以在查询中嵌入该函数的代码，或者在数据库中安装该函数。 用法选项有两种：临时使用和持久使用。 请参阅下面选项卡上的示例。

# <a name="ad-hoc"></a>[临时](#tab/adhoc)

如果是临时使用，请使用 [let 语句](../query/letstatement.md)嵌入该函数的代码。 不需要权限。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let series_fit_poly_fl=(tbl:(*), y_series:string, y_fit_series:string, fit_coeff:string, degree:int, x_series:string='', x_istime:bool=False)
{
    let kwargs = pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_coeff', fit_coeff, 'degree', degree, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_coeff = kargs["fit_coeff"]\n'
        'degree = kargs["degree"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'def fit(ts_row, x_col, y_col, deg):\n'
        '    y = ts_row[y_col]\n'
        '    # if x column exists check whether its a time column. If so, convert it to numeric seconds, else take it as is. If there is no x column creates sequential numbers\n'
        '    if x_col == "":\n'
        '       x = np.arange(len(y))\n'
        '    else:\n'
        '       if x_istime:\n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))/(1e9*60) #convert ticks to minutes\n'
        '           x = x - x.min()\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    coeff = np.polyfit(x, y, deg)\n'
        '    p = np.poly1d(coeff)\n'
        '    z = p(x)\n'
        '    return z, coeff\n'
        '\n'
        'result = df\n'
        'if len(df):\n'
        '   result[[y_fit_series, fit_coeff]] = df.apply(fit, axis=1, args=(x_series, y_series, degree,), result_type="expand")\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
};
//
// Fit 5th order polynomial to a regular (evenly spaced) time series, created with make-series
//
let max_t = datetime(2016-09-03);
demo_make_series1
| make-series num=count() on TimeStamp from max_t-1d to max_t step 5m by OsVer
| extend fnum = dynamic(null), coeff=dynamic(null), fnum1 = dynamic(null), coeff1=dynamic(null)
| invoke series_fit_poly_fl('num', 'fnum', 'coeff', 5)
| render timechart with(ycolumns=num, fnum)
```

# <a name="persistent"></a>[Persistent](#tab/persistent)

如果是永久使用，请使用 [.create 函数](../management/create-function.md)。  创建函数需要有[数据库用户权限](../management/access-control/role-based-authorization.md)。

### <a name="one-time-installation"></a>一次性安装

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
.create-or-alter function with (folder = "Packages\\Series", docstring = "Fit a polynomial of a specified degree to a series")
series_fit_poly_fl(tbl:(*), y_series:string, y_fit_series:string, fit_coeff:string, degree:int, x_series:string='', x_istime:bool=false)
{
    let kwargs = pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_coeff', fit_coeff, 'degree', degree, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_coeff = kargs["fit_coeff"]\n'
        'degree = kargs["degree"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'def fit(ts_row, x_col, y_col, deg):\n'
        '    y = ts_row[y_col]\n'
        '    # if x column exists check whether its a time column. If so, convert it to numeric seconds, else take it as is. If there is no x column creates sequential numbers\n'
        '    if x_col == "":\n'
        '       x = np.arange(len(y))\n'
        '    else:\n'
        '       if x_istime:\n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))/(1e9*60) #convert ticks to minutes\n'
        '           x = x - x.min()\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    coeff = np.polyfit(x, y, deg)\n'
        '    p = np.poly1d(coeff)\n'
        '    z = p(x)\n'
        '    return z, coeff\n'
        '\n'
        'result = df\n'
        'if len(df):\n'
        '   result[[y_fit_series, fit_coeff]] = df.apply(fit, axis=1, args=(x_series, y_series, degree,), result_type="expand")\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
}
```

### <a name="usage"></a>使用情况

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
//
// Fit 5th order polynomial to a regular (evenly spaced) time series, created with make-series
//
let max_t = datetime(2016-09-03);
demo_make_series1
| make-series num=count() on TimeStamp from max_t-1d to max_t step 5m by OsVer
| extend fnum = dynamic(null), coeff=dynamic(null), fnum1 = dynamic(null), coeff1=dynamic(null)
| invoke series_fit_poly_fl('num', 'fnum', 'coeff', 5)
| render timechart with(ycolumns=num, fnum)
```

---

:::image type="content" source="images/series-fit-poly-fl/usage-example.png" alt-text="图形，其中显示拟合到有规律的时序的第 5 阶多项式" border="false":::

## <a name="additional-examples"></a>其他示例

以下示例假定已安装该函数：

1. 测试不规律（间距不均匀）的时序
    
    <!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
    ```kusto
    //
    //  Test irregular (unevenly spaced) time series
    //
    let max_t = datetime(2016-09-03);
    demo_make_series1
    | where TimeStamp between ((max_t-2d)..max_t)
    | summarize num=count() by bin(TimeStamp, 5m), OsVer
    | order by TimeStamp asc
    | where hourofday(TimeStamp) % 6 != 0   //  delete every 6th hour to create unevenly spaced time series
    | summarize TimeStamp=make_list(TimeStamp), num=make_list(num) by OsVer
    | extend fnum = dynamic(null), coeff=dynamic(null)
    | invoke series_fit_poly_fl('num', 'fnum', 'coeff', 8, 'TimeStamp', True)
    | render timechart with(ycolumns=num, fnum)
    ```
    
    :::image type="content" source="images/series-fit-poly-fl/irregular-time-series.png" alt-text="图形，其中显示拟合到有规律的时序的第 5 阶多项式" border="false":::

1. x 轴和 y 轴上有干扰信息的第 5 阶多项式

    <!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
    ```kusto
    //
    // 5th order polynomial with noise on x & y axes
    //
    range x from 1 to 200 step 1
    | project x = rand()*5 - 2.3
    | extend y = pow(x, 5)-8*pow(x, 3)+10*x+6
    | extend y = y + (rand() - 0.5)*0.5*y
    | order by x asc 
    | summarize x=make_list(x), y=make_list(y)
    | extend y_fit = dynamic(null), coeff=dynamic(null)
    | invoke series_fit_poly_fl('y', 'y_fit', 'coeff', 5, 'x')
    |fork (project-away coeff) (project coeff | mv-expand coeff)
    | render linechart
    ```
        
    :::image type="content" source="images/series-fit-poly-fl/fifth-order-noise.png" alt-text="图形，其中显示拟合到有规律的时序的第 5 阶多项式":::
       
    :::image type="content" source="images/series-fit-poly-fl/fifth-order-noise-table.png" alt-text="图形，其中显示拟合到有规律的时序的第 5 阶多项式" border="false":::
