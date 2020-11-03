---
title: array_split() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 array_split()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/28/2018
ms.date: 10/29/2020
ms.openlocfilehash: 636a0e2ff725242b8e834e71192cb5582522e3d6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106167"
---
# <a name="array_split"></a>array_split()

根据拆分索引将数组拆分成多个数组，并将生成的数组打包到一个动态数组中。

## <a name="syntax"></a>语法

`array_split`( *`arr`* , *`indices`* )

## <a name="arguments"></a>参数

* *`arr`* ：要拆分的输入数组，必须是动态数组。
* *`indices`* ：具有拆分索引的整数或动态整数数组（从零开始），负值将转换为 array_length + 值。

## <a name="returns"></a>返回

包含 N+1 个数组的动态数组，其值在 `arr` 中 `[0..i1), [i1..i2), ... [iN..array_length)` 的范围内，其中 N 是输入索引的数目，`i1...iN` 是索引。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend arr_split=array_split(arr, 2)
```

|`arr`|`arr_split`|
|---|---|
|[1,2,3,4,5]|[[1,2],[3,4,5]]|

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend arr_split=array_split(arr, dynamic([1,3]))
```

|`arr`|`arr_split`|
|---|---|
|[1,2,3,4,5]|[[1],[2,3],[4,5]]|
