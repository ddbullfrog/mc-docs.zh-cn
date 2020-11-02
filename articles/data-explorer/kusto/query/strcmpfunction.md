---
title: strcmp() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 strcmp()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: 7e74768c95ee53c6bd2a501c5880be17b69cfc60
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103710"
---
# <a name="strcmp"></a>strcmp()

比较两个字符串。

此函数从比较每个字符串的第一个字符开始。 如果第一个字符相同，则继续比较接下来的字符对，直到字符不同或到达较短字符串的末尾。

## <a name="syntax"></a>语法

`strcmp(`*string1*`,` *string2*`)` 

## <a name="arguments"></a>参数

* string1：要比较的第一个输入字符串。 
* string2：要比较的第二个输入字符串。

## <a name="returns"></a>返回

返回一个整数值，指示字符串之间的关系：
* <0 - 第一个不匹配的字符在 string1 中的值小于 string2 中的值
* 0 - 两个字符串的内容相同
* >0 - 第一个不匹配的字符在 string1 中的值大于 string2 中的值

## <a name="examples"></a>示例

```
datatable(string1:string, string2:string)
["ABC","ABC",
"abc","ABC",
"ABC","abc",
"abcde","abc"]
| extend result = strcmp(string1,string2)
```

|string1|string2|result|
|---|---|---|
|ABC|ABC|0|
|abc|ABC|1|
|ABC|abc|-1|
|abcde|abc|1|