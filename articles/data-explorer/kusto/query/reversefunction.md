---
title: reverse() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 reverse()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 0069a9de66aca540c369b6cf462aaa92d3062a4c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105015"
---
# <a name="reverse"></a>reverse()

此函数会反转输入字符串的顺序。
如果输入值不是 `string` 类型，则函数会强制将值转换为 `string` 类型。

## <a name="syntax"></a>语法

`reverse(`*source*`)`

## <a name="arguments"></a>参数

* source：输入值。  

## <a name="returns"></a>返回

字符串值的逆序排序形式。

## <a name="examples"></a>示例

```kusto
print str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
| extend rstr = reverse(str)
```

|str|rstr|
|---|---|
|ABCDEFGHIJKLMNOPQRSTUVWXYZ|ZYXWVUTSRQPONMLKJIHGFEDCBA|


```kusto
print ['int'] = 12345, ['double'] = 123.45, 
['datetime'] = datetime(2017-10-15 12:00), ['timespan'] = 3h
| project rint = reverse(['int']), rdouble = reverse(['double']), 
rdatetime = reverse(['datetime']), rtimespan = reverse(['timespan'])
```

|rint|rdouble|rdatetime|rtimespan|
|---|---|---|---|
|54321|54.321|Z0000000.00:00:21T51-01-7102|00:00:30|
