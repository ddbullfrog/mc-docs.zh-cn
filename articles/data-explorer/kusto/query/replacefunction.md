---
title: replace() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 replace()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: f36833fb0d739cc0c1fd5f9ce741ac15df3da04f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105020"
---
# <a name="replace"></a>replace()

将所有正则表达式匹配项替换为其他字符串。

## <a name="syntax"></a>语法

`replace(`regex`,` rewrite`,` text`)`  

## <a name="arguments"></a>参数

* regex：用于搜索 text 的[正则表达式](https://github.com/google/re2/wiki/Syntax)。 可在“(括号)”中包含捕获组。 
* rewrite：matchingRegex 执行的任何匹配的替换正则表达式。 使用 `\0` 引用整个匹配项，`\1` 用于第一个捕获组，`\2` 等用于后续捕获组。
* *text* ：一个字符串。

## <a name="returns"></a>返回

使用 *rewrite* 计算结果替换 *regex* 的所有匹配项后的 *text* 。 匹配项不重叠。

## <a name="example"></a>示例

此语句：

```kusto
range x from 1 to 5 step 1
| extend str=strcat('Number is ', tostring(x))
| extend replaced=replace(@'is (\d+)', @'was: \1', str)
```

具有以下结果：

| x    | str | 替换的内容|
|---|---|---|
| 1    | 数值为 1.000000  | 数值曾为：1.000000|
| 2    | 数值为 2.000000  | 数值曾为：2.000000|
| 3    | 数值为 3.000000  | 数值曾为：3.000000|
| 4    | 数值为 4.000000  | 数值曾为：4.000000|
| 5    | 数值为 5.000000  | 数值曾为：5.000000|
 