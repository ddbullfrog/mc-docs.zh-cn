---
title: tdigest_merge() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 tdigest_merge()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/09/2019
ms.date: 09/30/2020
ms.openlocfilehash: 98953cf7fa021778cdc8836848ff6b98b0d52355
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103986"
---
# <a name="tdigest_merge"></a>tdigest_merge()

合并 `tdigest` 结果（聚合版 [`tdigest_merge()`](tdigest-merge-aggfunction.md) 的标量版本）。

在[此处](percentiles-aggfunction.md#estimation-error-in-percentiles)详细了解基础算法 (T-Digest) 和预估误差。

## <a name="syntax"></a>语法

`merge_tdigests(` *Expr1*`,` *Expr2*`, ...)`

`tdigest_merge(` *Expr1*`,` *Expr2*`, ...)` - 别名。

## <a name="arguments"></a>参数

* 列，包含要合并的 `tdigest` 值。

## <a name="returns"></a>返回

将列 `*Expr1*`、`*Expr2*`、... `*ExprN*` 合并为一个 `tdigest` 后的结果。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 10 step 1 
| extend y = x + 10
| summarize tdigestX = tdigest(x), tdigestY = tdigest(y)
| project merged = tdigest_merge(tdigestX, tdigestY)
| project percentile_tdigest(merged, 100, typeof(long))
```

|percentile_tdigest_merged|
|---|
|20|
