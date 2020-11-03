---
title: set_union() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 set_union()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 06/02/2019
ms.date: 09/30/2020
ms.openlocfilehash: 06ac538be59bb891e2d751cc59d91caf149d15a1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106028"
---
# <a name="set_union"></a>set_union()

返回一个 `dynamic` 数组，其中包含在任意数组 (arr1 ∪ arr2 ∪ ...) 中的所有非重复值的集合。

## <a name="syntax"></a>语法

`set_union(`*arr1*`, `*arr2*`[`,` *arr3*, ...]``)`

## <a name="arguments"></a>参数

* arr1...arrN：用于创建并集的输入数组（至少两个数组）。 所有参数都必须是动态数组（请参阅 [pack_array](packarrayfunction.md)）。 

## <a name="returns"></a>返回

返回一个动态数组，其中包含在任意数组中的所有非重复值的集合。 请参阅 [`set_intersect()`](setintersectfunction.md) 和 [`set_difference()`](setdifferencefunction.md)。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| extend w = z * 2
| extend a1 = pack_array(x,y,x,z), a2 = pack_array(x, y), a3 = pack_array(w)
| project set_union(a1, a2, a3)
```

|Column1|
|---|
|[1,2,4,8]|
|[2,4,8,16]|
|[3,6,12,24]|
