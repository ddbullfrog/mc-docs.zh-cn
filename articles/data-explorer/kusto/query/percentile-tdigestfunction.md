---
title: percentile_tdigest() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 percentile_tdigest()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/10/2019
ms.date: 10/29/2020
ms.openlocfilehash: 1b96e105e0c74e22404c750e784edc869d945324
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105913"
---
# <a name="percentile_tdigest"></a>percentile_tdigest()

根据 `tdigest` 结果（已由 [tdigest()](tdigest-aggfunction.md) 或 [tdigest_merge()](tdigest-merge-aggfunction.md) 生成）计算百分比结果。

## <a name="syntax"></a>语法

`percentile_tdigest(`*`Expr`* `,` *Percentile1* [`,` *typeLiteral* ]`)`

`percentiles_array_tdigest(`*`Expr`* `,` *Percentile1* [`,` *Percentile2* ] ...[`,` *PercentileN* ]`)`

`percentiles_array_tdigest(`*`Expr`* `,` *动态数组*`)`

## <a name="arguments"></a>参数

* Expr：已由 [`tdigest`](tdigest-aggfunction.md) 或 [tdigest_merge()](tdigest-merge-aggfunction.md) 生成的表达式。
* Percentile 是一个双精度常量，用于指定百分位数。
* typeLiteral：可选的类型文本（例如 `typeof(long)`）。 如果提供，结果集将为此类型。 
* 动态数组：整数或浮点数动态数组中的百分位数列表。

## <a name="returns"></a>返回

`Expr` 中每个值的百分位数/percentilesw 值。

**提示**

* 此函数必须至少接收一个百分比（或者可能更多，请参阅上面的语法：Percentile1 [`,` Percentile2] ...[`,` PercentileN]），结果将是一个包含结果的动态数组。 （例如 [`percentiles()`](percentiles-aggfunction.md)）
  
* 如果仅提供了一个百分比，并且还提供了类型，则结果将是一个为其提供了该百分比的结果的同一类型的列。 在这种情况下，所有 `tdigest` 函数都必须为该类型。

* 如果 `Expr` 包含不同类型的 `tdigest` 函数，请不要提供类型。 结果将为动态类型。 请参阅以下示例。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| project percentile_tdigest(tdigestRes, 100, typeof(int))
```

|percentile_tdigest_tdigestRes|
|---|
|0|
|62000000|
|110000000|
|1200000|
|250000|

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| project percentiles_array_tdigest(tdigestRes, range(0, 100, 50), typeof(int))
```

|percentile_tdigest_tdigestRes|
|---|
|[0,0,0]|
|[0,0,62000000]|
|[0,0,110000000]|
|[0,0,1200000]|
|[0,0,250000]|

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| union (StormEvents | summarize tdigestRes = tdigest(EndTime) by State)
| project percentile_tdigest(tdigestRes, 100)
```

|percentile_tdigest_tdigestRes|
|---|
|[0]|
|[62000000]|
|["2007-12-20T11:30:00.0000000Z"]|
|["2007-12-31T23:59:00.0000000Z"]|
