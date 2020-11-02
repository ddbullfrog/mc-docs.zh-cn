---
title: trim() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 trim()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: b4e470d09171e101f5481ef7399ac415ff527e41
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105494"
---
# <a name="trim"></a>trim()

删除指定正则表达式的所有前导匹配项和尾随匹配项。

## <a name="syntax"></a>语法

`trim(`*regex*`,` *text*`)`

## <a name="arguments"></a>参数

* regex：要从 text 的开头和/或结尾修剪的字符串或[正则表达式](re2.md)。  
* *text* ：一个字符串。

## <a name="returns"></a>返回

修剪 text 开头和/或结尾的 regex 匹配项后的 text。

## <a name="example"></a>示例

以下语句从 string_to_trim 的开头和结尾修剪 substring：

```kusto
let string_to_trim = @"--https://bing.com--";
let substring = "--";
print string_to_trim = string_to_trim, trimmed_string = trim(substring,string_to_trim)
```

|string_to_trim|trimmed_string|
|---|---|
|--https://bing.com--|https://bing.com|

下一条语句从字符串的开头和结尾修剪所有非单词字符：

```kusto
range x from 1 to 5 step 1
| project str = strcat("-  ","Te st",x,@"// $")
| extend trimmed_str = trim(@"[^\w]+",str)
```

|str|trimmed_str|
|---|---|
|-  Te st1// $|Te st1|
|-  Te st2// $|Te st2|
|-  Te st3// $|Te st3|
|-  Te st4// $|Te st4|
|-  Te st5// $|Te st5|


 