---
title: bag_remove_keys() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 bag_remove_keys()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/04/2020
ms.date: 10/30/2020
ms.openlocfilehash: ae7fa662c9d6a6086c182329fb5d93680c10a9da
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106694"
---
# <a name="bag_remove_keys"></a>bag_remove_keys()

从 `dynamic` 属性包中删除键和关联的值。

## <a name="syntax"></a>语法

`bag_remove_keys(`*bag*`, `*keys*`)`

## <a name="arguments"></a>参数

* *bag* ：`dynamic` 属性包输入。
* *keys* ：`dynamic` 数组包含要从输入中删除的键。 键指的是属性包的第一级。
不支持在嵌套级别上指定键。

## <a name="returns"></a>返回

返回不带指定键及其值的 `dynamic` 属性包。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(input:dynamic)
[
    dynamic({'key1' : 123,     'key2': 'abc'}),
    dynamic({'key1' : 'value', 'key3': 42.0}),
]
| extend result=bag_remove_keys(input, dynamic(['key2', 'key4']))
```

|input|result|
|---|---|
|{<br>  "key1":123,<br>  "key2": "abc"<br>}|{<br>  "key1":123<br>}|
|{<br>  "key1": "value",<br>  "key3":42.0<br>}|{<br>  "key1": "value",<br>  "key3":42.0<br>}|
