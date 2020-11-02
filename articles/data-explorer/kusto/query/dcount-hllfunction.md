---
title: dcount_hll() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 dcount_hll()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/15/2019
ms.date: 10/29/2020
ms.openlocfilehash: ab9a9874110aa111172db5ef275d37070e687302
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104268"
---
# <a name="dcount_hll"></a>dcount_hll()

根据 hll 结果（已由 [hll](hll-aggfunction.md) 或 [hll_merge](hll-merge-aggfunction.md) 生成）计算 dcount。

阅读[基础算法 (HyperLogLog  ) 和估算准确度](dcount-aggfunction.md#estimation-accuracy)。

## <a name="syntax"></a>语法

`dcount_hll(`*Expr*`)`

## <a name="arguments"></a>参数

* Expr：已由 [hll](hll-aggfunction.md) 或 [hll-merge](hll-merge-aggfunction.md) 生成的表达式

## <a name="returns"></a>返回

Expr 中每个值的非重复计数

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| summarize hllRes = hll(DamageProperty) by bin(StartTime,10m)
| summarize hllMerged = hll_merge(hllRes)
| project dcount_hll(hllMerged)
```

|dcount_hll_hllMerged|
|---|
|315|
