---
title: percentrank_tdigest() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 percentrank_tdigest()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/10/2019
ms.date: 10/29/2020
ms.openlocfilehash: ca65719582ffe0cf77446ad0e9446cfe83893014
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105030"
---
# <a name="percentrank_tdigest"></a>percentrank_tdigest()

计算集合中值的近似排序，其中排序表示为集大小的百分比。
此函数可以看作是百分位数的倒数。

## <a name="syntax"></a>语法

`percentrank_tdigest(`*TDigest*`,` *Expr*`)`

## <a name="arguments"></a>参数

* TDigest：[tdigest()](tdigest-aggfunction.md) 或 [tdigest_merge()](tdigest-merge-aggfunction.md) 生成的表达式。
* Expr：表示要用于百分比排序计算的值的表达式。

## <a name="returns"></a>返回

数据集中的值的百分比排序。

**提示**

1) 第二个参数的类型和 `tdigest` 中元素的类型应相同。

2) 第一个参数应是 [tdigest()](tdigest-aggfunction.md) 或 [tdigest_merge()](tdigest-merge-aggfunction.md) 生成的 TDigest

## <a name="examples"></a>示例

获得价值 4490$ 的财产损失的 percentrank_tdigest() 约为 85%：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentrank_tdigest(tdigestRes, 4490)

```

|Column1|
|---|
|85.0015237192293|


对财产损失使用百分位数 85 应得出 4490$：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentile_tdigest(tdigestRes, 85, typeof(long))

```

|percentile_tdigest_tdigestRes|
|---|
|4490|
