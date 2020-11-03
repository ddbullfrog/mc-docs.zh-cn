---
title: Array_index_of() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 array_index_of()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 01/22/2020
ms.date: 10/29/2020
ms.openlocfilehash: 59e3bc70dd51ff2015a700c575a40440df66f611
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106170"
---
# <a name="array_index_of"></a>array_index_of()

搜索指定项的数组，并返回其位置。

## <a name="syntax"></a>语法

`array_index_of(`*array* , *value*`)`

## <a name="arguments"></a>参数

* *array* ：输入要搜索的数组。
* *value* ：要搜索的值。 此值的类型应为长型、整数型、双精度型、日期/时间型、时间范围型、十进制型、字符串型或 guid。

## <a name="returns"></a>返回

查找的从零开始的索引位置。
如果在数组中找不到该值，则返回 -1。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=array_index_of(arr, "example")
```

|结果|
|---|
|3|

## <a name="see-also"></a>另请参阅

如果仅希望检查数组中是否存在某个值，但对其位置不感兴趣，可以使用 [set_has_element(`arr`, `value`)](sethaselementfunction.md)。 此函数将提高查询的可读性。 这两个函数具有相同的性能。
