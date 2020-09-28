---
title: dcount_intersect 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 dcount_intersect 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/24/2020
ms.openlocfilehash: a485fd89de22ae7968017e0039187a849774892d
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146782"
---
# <a name="dcount_intersect-plugin"></a>dcount_intersect 插件

基于 `hll` 值（N 在 [2..16] 范围内）计算 N 个集合之间的交集，并返回 N 个 `dcount` 值。

给定集 S<sub>1</sub>、S<sub>2</sub>... S<sub>n</sub> - 返回值将代表以下各项的非重复计数：  
S<sub>1</sub>、S<sub>1</sub> ∩ S<sub>2</sub>、  
S<sub>1</sub> ∩ S<sub>2</sub> ∩ S<sub>3</sub>、  
... ,  
S<sub>1</sub> ∩ S<sub>2</sub> ∩ ... ∩ S<sub>n</sub>

```kusto
T | evaluate dcount_intersect(hll_1, hll_2, hll_3)
```

## <a name="syntax"></a>语法

*T* `| evaluate` `dcount_intersect(`*hll_1*, *hll_2*, [`,` *hll_3*`,` ...]`)`

## <a name="arguments"></a>参数

* *T*：输入表格表达式。
* hll_i：使用 [`hll()`](./hll-aggfunction.md) 函数计算的集 S<sub>i</sub> 的值。

## <a name="returns"></a>返回

返回一个表，其中包含 N 个 `dcount` 值（按列，表示交集）。
列名为 s0、s1...（直到 n-1）。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
// Generate numbers from 1 to 100
range x from 1 to 100 step 1
| extend isEven = (x % 2 == 0), isMod3 = (x % 3 == 0), isMod5 = (x % 5 == 0)
// Calculate conditional HLL values (note that '0' is included in each of them as additional value, so we will subtract it later)
| summarize hll_even = hll(iif(isEven, x, 0), 2),
            hll_mod3 = hll(iif(isMod3, x, 0), 2),
            hll_mod5 = hll(iif(isMod5, x, 0), 2) 
// Invoke the plugin that calculates dcount intersections         
| evaluate dcount_intersect(hll_even, hll_mod3, hll_mod5)
| project evenNumbers = s0 - 1,             //                             100 / 2 = 50
          even_and_mod3 = s1 - 1,           // gcd(2,3) = 6, therefor:     100 / 6 = 16
          even_and_mod3_and_mod5 = s2 - 1   // gcd(2,3,5) is 30, therefore: 100 / 30 = 3 
```

|evenNumbers|even_and_mod3|even_and_mod3_and_mod5|
|---|---|---|
|50|16|3|