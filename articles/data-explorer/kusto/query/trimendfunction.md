---
title: trim_end() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 trim_end()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: b8b5fb2108783294d7785b30293c44af1a15d435
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105083"
---
# <a name="trim_end"></a>trim_end()

删除指定正则表达式的尾随匹配项。

## <a name="syntax"></a>语法

`trim_end(`*regex*`,` *text*`)`

## <a name="arguments"></a>参数

* regex：要从 text 的结尾修剪的字符串或[正则表达式](re2.md)。  
* *text* ：一个字符串。

## <a name="returns"></a>返回

修剪 text 结尾的 regex 匹配项后的 text。

## <a name="example"></a>示例

以下语句从 string_to_trim 的结尾修剪 substring：

```kusto
let string_to_trim = @"bing.com";
let substring = ".com";
print string_to_trim = string_to_trim,trimmed_string = trim_end(substring,string_to_trim)
```

|string_to_trim|trimmed_string|
|--------------|--------------|
|bing.com      |bing          |

下一条语句从字符串的结尾修剪所有非单词字符：

```kusto
print str = strcat("-  ","Te st",x,@"// $")
| extend trimmed_str = trim_end(@"[^\w]+",str)
```

|str          |trimmed_str|
|-------------|-----------|
|-  Te st1// $|-  Te st1  |
|-  Te st2// $|-  Te st2  |
|-  Te st3// $|-  Te st3  |
|-  Te st4// $|-  Te st4  |
|-  Te st5// $|-  Te st5  |