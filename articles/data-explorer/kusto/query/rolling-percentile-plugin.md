---
title: rolling_percentile 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 rolling_percentile 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 04bdf9a6079bf5b61d87f8384d64337b2a381a00
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105013"
---
# <a name="rolling_percentile-plugin"></a>rolling_percentile() 插件

按 BinSize 在 BinsPerWindow 大小的滚动（滑动）窗口中，返回 ValueColumn 填充值的指定百分位数的估计值  。

```kusto
T | evaluate rolling_percentile(ValueColumn, Percentile, IndexColumn, BinSize, BinsPerWindow)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `rolling_percentile(`*ValueColumn*`,` *Percentile*`,` *IndexColumn*`,` *BinSize*`,` *BinsPerWindow*  [`,` *dim1*`,` *dim2*`,` ...] `)`

## <a name="arguments"></a>参数

* *T* ：输入表格表达式。
* *ValueColumn* ：列的名称，其中包含要计算百分位数的值。 
* *Percentile* ：带有要计算的百分位数的标量。
* *IndexColumn* ：要对其运行滚动窗口的列名称。
* BinSize：标量，带有要在 IndexColumn 上应用的箱大小。
* *BinsPerWindow* ：标量，具有每个窗口中包含的箱数。
* *dim1* , *dim2* , ... ：（可选）要作为切片依据的维度列的列表。

## <a name="returns"></a>返回

返回一个表，其中为每个箱设有一行（如果指定，则为维度的组合），该表含有截至箱处的窗口中的滚动百分位值。 每个时间窗口的非重复计数值、新值的非重复计数、聚合非重复计数。

输出表架构如下：


|IndexColumn|dim1|...|dim_n|rolling_BinsPerWindow_percentile_ValueColumn_Pct
|---|---|---|---|---|


## <a name="examples"></a>示例

### <a name="rolling-3-day-median-value-per-day"></a>每天滚动 3 天的中值 

下一个查询按每日粒度计算 3 天的中值。 输出中的每一行都表示过去 3 箱（天）的中值，包括箱本身。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let T = 
range idx from 0 to 24*10-1 step 1
| project Timestamp = datetime(2018-01-01) + 1h*idx, val=idx+1
| extend EvenOrOdd = iff(val % 2 == 0, "Even", "Odd");
 T  
 | evaluate rolling_percentile(val, 50, Timestamp, 1d, 3)
```

|Timestamp|rolling_3_percentile_val_50|
|---|---|
|2018-01-01 00:00:00.0000000|   12|
|2018-01-02 00:00:00.0000000|   24|
|2018-01-03 00:00:00.0000000|   36|
|2018-01-04 00:00:00.0000000|   60|
|2018-01-05 00:00:00.0000000|   84|
|2018-01-06 00:00:00.0000000|   108|
|2018-01-07 00:00:00.0000000|   132|
|2018-01-08 00:00:00.0000000|   156|
|2018-01-09 00:00:00.0000000|   180|
|2018-01-10 00:00:00.0000000|   204|

### <a name="rolling-3-day-median-value-per-day-by-dimension"></a>每天按维度滚动 3 天的中值

依然采用上面的示例，但现在还要计算针对维度的每个值进行分区的滚动窗口。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let T = 
range idx from 0 to 24*10-1 step 1
| project Timestamp = datetime(2018-01-01) + 1h*idx, val=idx+1
| extend EvenOrOdd = iff(val % 2 == 0, "Even", "Odd");
 T  
 | evaluate rolling_percentile(val, 50, Timestamp, 1d, 3, EvenOrOdd)
```

|Timestamp| EvenOrOdd|  rolling_3_percentile_val_50|
|---|---|---|
|2018-01-01 00:00:00.0000000|   偶数|   12|
|2018-01-02 00:00:00.0000000|   偶数|   24|
|2018-01-03 00:00:00.0000000|   偶数|   36|
|2018-01-04 00:00:00.0000000|   偶数|   60|
|2018-01-05 00:00:00.0000000|   偶数|   84|
|2018-01-06 00:00:00.0000000|   偶数|   108|
|2018-01-07 00:00:00.0000000|   偶数|   132|
|2018-01-08 00:00:00.0000000|   偶数|   156|
|2018-01-09 00:00:00.0000000|   偶数|   180|
|2018-01-10 00:00:00.0000000|   偶数|   204|
|2018-01-01 00:00:00.0000000|   奇数|    11|
|2018-01-02 00:00:00.0000000|   奇数|    23|
|2018-01-03 00:00:00.0000000|   奇数|    35|
|2018-01-04 00:00:00.0000000|   奇数|    59|
|2018-01-05 00:00:00.0000000|   奇数|    83|
|2018-01-06 00:00:00.0000000|   奇数|    107|
|2018-01-07 00:00:00.0000000|   奇数|    131|
|2018-01-08 00:00:00.0000000|   奇数|    155|
|2018-01-09 00:00:00.0000000|   奇数|    179|
|2018-01-10 00:00:00.0000000|   奇数|    203|
