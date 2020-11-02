---
title: quantize_fl() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中用户定义的函数 quantize_fl()。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/08/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1da43a6f89c48578c1cf90d9cf1498e5434e3cbc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105111"
---
# <a name="quantize_fl"></a>quantize_fl()


函数 `quantize_fl()` 对指标列进行分箱。 它基于 K-Means 算法将指标列量化到类别标签。

> [!NOTE]
> * `quantize_fl()` 是 [UDF（用户定义的函数）](../query/functions/user-defined-functions.md)。
> * 此函数包含内联 Python，需要在群集上[启用 python() 插件](../query/pythonplugin.md#enable-the-plugin)。 有关详细信息，请参阅[用法](#usage)。

## <a name="syntax"></a>语法

`T | invoke quantize_fl(`*num_bins*`,` *in_cols*`,` *out_cols*`,` *labels*`)`

## <a name="arguments"></a>参数

* num_bins：所需的箱数。
* in_cols：动态数组，其中包含要量化的列的名称。
* out_cols：动态数组，其中包含分箱值各自的输出列的名称。
* labels：包含标签名称的动态数组。 此参数是可选的。 如果未提供 *Labels* ，将使用箱范围。

## <a name="usage"></a>使用情况

`quantize_fl()` 是用户定义的[表格函数](../query/functions/user-defined-functions.md#tabular-function)，需使用 [invoke 运算符](../query/invokeoperator.md)进行应用。 可以在查询中嵌入其代码，或将其安装在数据库中。 用法选项有两种：临时使用和永久使用。 请参阅下面选项卡上的示例。

# <a name="ad-hoc"></a>[临时](#tab/adhoc)

如果是临时使用，请使用 [let 语句](../query/letstatement.md)嵌入其代码。 不需要权限。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let quantize_fl=(tbl:(*), num_bins:int, in_cols:dynamic, out_cols:dynamic, labels:dynamic=dynamic(null))
{
    let kwargs = pack('num_bins', num_bins, 'in_cols', in_cols, 'out_cols', out_cols, 'labels', labels);
    let code =
        '\n'
        'from sklearn.preprocessing import KBinsDiscretizer\n'
        '\n'
        'num_bins = kargs["num_bins"]\n'
        'in_cols = kargs["in_cols"]\n'
        'out_cols = kargs["out_cols"]\n'
        'labels = kargs["labels"]\n'
        '\n'
        'result = df\n'
        'binner = KBinsDiscretizer(n_bins=num_bins, encode="ordinal", strategy="kmeans")\n'
        'df_in = df[in_cols]\n'
        'bdata = binner.fit_transform(df_in)\n'
        'if labels is None:\n'
        '    for i in range(len(out_cols)):    # loop on each column and convert it to binned labels\n'
        '        ii = np.round(binner.bin_edges_[i], 3)\n'
        '        labels = [str(ii[j-1]) + \'-\' + str(ii[j]) for j in range(1, num_bins+1)]\n'
        '        result.loc[:,out_cols[i]] = np.take(labels, bdata[:, i].astype(int))\n'
        'else:\n'
        '    result[out_cols] = np.take(labels, bdata.astype(int))\n'
        ;
    tbl
    | evaluate python(typeof(*), code, kwargs)
};
//
union 
(range x from 1 to 5 step 1),
(range x from 10 to 15 step 1),
(range x from 20 to 25 step 1)
| extend x_label='', x_bin=''
| invoke quantize_fl(3, pack_array('x'), pack_array('x_label'), pack_array('Low', 'Med', 'High'))
| invoke quantize_fl(3, pack_array('x'), pack_array('x_bin'), dynamic(null))
```

# <a name="persistent"></a>[Persistent](#tab/persistent)

如果是永久使用，请使用 [.create 函数](../management/create-function.md)。 创建函数需要有[数据库用户权限](../management/access-control/role-based-authorization.md)。

### <a name="one-time-installation"></a>一次性安装

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
.create function with (folder = "Packages\\ML", docstring = "Binning metric columns")
quantize_fl(tbl:(*), num_bins:int, in_cols:dynamic, out_cols:dynamic, labels:dynamic)
{
    let kwargs = pack('num_bins', num_bins, 'in_cols', in_cols, 'out_cols', out_cols, 'labels', labels);
    let code =
        '\n'
        'from sklearn.preprocessing import KBinsDiscretizer\n'
        '\n'
        'num_bins = kargs["num_bins"]\n'
        'in_cols = kargs["in_cols"]\n'
        'out_cols = kargs["out_cols"]\n'
        'labels = kargs["labels"]\n'
        '\n'
        'result = df\n'
        'binner = KBinsDiscretizer(n_bins=num_bins, encode="ordinal", strategy="kmeans")\n'
        'df_in = df[in_cols]\n'
        'bdata = binner.fit_transform(df_in)\n'
        'if labels is None:\n'
        '    for i in range(len(out_cols)):    # loop on each column and convert it to binned labels\n'
        '        ii = np.round(binner.bin_edges_[i], 3)\n'
        '        labels = [str(ii[j-1]) + \'-\' + str(ii[j]) for j in range(1, num_bins+1)]\n'
        '        result.loc[:,out_cols[i]] = np.take(labels, bdata[:, i].astype(int))\n'
        'else:\n'
        '    result[out_cols] = np.take(labels, bdata.astype(int))\n'
        ;
    tbl
    | evaluate python(typeof(*), code, kwargs)
}
```

### <a name="usage"></a>使用情况

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
union 
(range x from 1 to 5 step 1),
(range x from 10 to 15 step 1),
(range x from 20 to 25 step 1)
| extend x_label='', x_bin=''
| invoke quantize_fl(3, pack_array('x'), pack_array('x_label'), pack_array('Low', 'Med', 'High'))
| invoke quantize_fl(3, pack_array('x'), pack_array('x_bin'), dynamic(null))
```

---

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
x    x_label    x_bin
1    Low        1.0-7.75
2    Low        1.0-7.75
3    Low        1.0-7.75
4    Low        1.0-7.75
5    Low        1.0-7.75
20   High       17.5-25.0
21   High       17.5-25.0
22   High       17.5-25.0
23   High       17.5-25.0
24   High       17.5-25.0
25   High       17.5-25.0
10   Med        7.75-17.5
11   Med        7.75-17.5
12   Med        7.75-17.5
13   Med        7.75-17.5
14   Med        7.75-17.5
15   Med        7.75-17.5
```
