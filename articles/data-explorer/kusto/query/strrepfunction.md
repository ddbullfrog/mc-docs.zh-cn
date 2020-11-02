---
title: strrep() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 strrep()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 08e23afc02d1d3d9a5ff1ee329f03b4c5fee68b8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103703"
---
# <a name="strrep"></a>strrep()

将给定[字符串](./scalar-data-types/string.md)重复提供的次数。

* 如果第一个或第三个参数不是字符串类型，则会将其强制转换为字符串。

## <a name="syntax"></a>语法

`strrep(`*value* , *multiplier* ,[ *delimiter* ]`)`

## <a name="arguments"></a>参数

* value：输入表达式
* multiplier：正整数值（从 1 到 1024）
* delimiter：可选字符串表达式（默认值：空字符串）

## <a name="returns"></a>返回

重复指定次数的值，使用 delimiter 进行连接。

如果 multiplier 大于允许的最大值 (1024)，则输入字符串将重复 1024 次。
 
## <a name="example"></a>示例

```kusto
print from_str = strrep('ABC', 2), from_int = strrep(123,3,'.'), from_time = strrep(3s,2,' ')
```

|from_str|from_int|from_time|
|---|---|---|
|ABCABC|123.123.123|00:00:03 00:00:03|