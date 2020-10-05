---
title: series_moving_avg_fl() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的用户定义函数 series_moving_avg_fl()。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/08/2020
ms.date: 09/24/2020
ms.openlocfilehash: fb6543bebeb037b62a4107372b4661fb71d26754
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146876"
---
# <a name="series_moving_avg_fl"></a>series_moving_avg_fl()

对序列应用移动平均滤波器。

函数 `series_moving_avg_fl()` 接受一个包含动态数值数组的表达式作为输入，并应用[简单移动平均](https://en.wikipedia.org/wiki/Moving_average#Simple_moving_average)滤波器。

> [!NOTE]
> 此函数是 [UDF（用户定义的函数）](../query/functions/user-defined-functions.md)。 有关详细信息，请参阅[用法](#usage)。

## <a name="syntax"></a>语法

`series_moving_avg_fl(`*y_series*`,` *n*`, [`*center*`])`
  
## <a name="arguments"></a>参数

* y_series：数值的动态数组单元。
* n：移动平均滤波器的宽度。
* center：一个可选布尔值，该值指示移动平均是否为以下选项之一：
    * 在当前点前后对称地应用于窗口，或 
    * 从当前点向后应用于窗口。 <br>
    默认情况下，center 为 False。

## <a name="usage"></a>使用情况

`series_moving_avg_fl()` 是用户定义的函数。 可以在查询中嵌入其代码，或将其安装在数据库中。 用法选项有两种：临时使用和永久使用。 有关示例，请参阅下面的选项卡。

# <a name="ad-hoc"></a>[临时](#tab/adhoc)

如果是临时使用，请使用 [let 语句](../query/letstatement.md)嵌入其代码。 不需要权限。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let series_moving_avg_fl = (y_series:dynamic, n:int, center:bool=false)
{
    series_fir(y_series, repeat(1, n), true, center)
}
;
//
//  Moving average of 5 bins
//
demo_make_series1
| make-series num=count() on TimeStamp step 1h by OsVer
| extend num_ma=series_moving_avg_fl(num, 5, True)
| render timechart 
```

# <a name="persistent"></a>[Persistent](#tab/persistent)

如果是永久使用，请使用 [.create 函数](../management/create-function.md)。 创建函数需要[数据库用户权限](../management/access-control/role-based-authorization.md)。

### <a name="one-time-installation"></a>一次性安装

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
.create-or-alter function with (folder = "Packages\\Series", docstring = "Calculate moving average of specified width")
series_moving_avg_fl(y_series:dynamic, n:int, center:bool=false)
{
    series_fir(y_series, repeat(1, n), true, center)
}
```

### <a name="usage"></a>使用情况

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
//
//  Moving average of 5 bins
//
demo_make_series1
| make-series num=count() on TimeStamp step 1h by OsVer
| extend num_ma=series_moving_avg_fl(num, 5, True)
| render timechart 
```

---

:::image type="content" source="images/series-moving-avg-fl/moving-average-five-bins.png" alt-text="描述 5 箱的移动平均的图" border="false":::
