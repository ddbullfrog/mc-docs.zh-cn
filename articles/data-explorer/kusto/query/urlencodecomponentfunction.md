---
title: url_encode_component() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 url_encode_component()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/17/2020
ms.date: 09/30/2020
ms.openlocfilehash: f52fcc0de9750a1795bd3fffd02112fca33b1fb6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106180"
---
# <a name="url_encode_component"></a>url_encode_component()

此函数将输入 URL 的字符转换为可通过 Internet 传输的格式。 

与 [url_encode](./urlencodefunction.md) 不同的是，它将空格编码为“20%”而不是“+”。

## <a name="syntax"></a>语法

`url_encode_component(`url`)`

## <a name="arguments"></a>参数

* url：输入 URL（字符串）。  

## <a name="returns"></a>返回

URL（字符串），已转换为可通过 Internet 传输的格式。

## <a name="examples"></a>示例

```kusto
let url = @'https://www.bing.com/hello word/';
print original = url, encoded = url_encode_component(url)
```

|原配|已编码|
|---|---|
|https://www.bing.com/hello word/|https%3a%2f%2f www.bing.com%2fhello%20word|


 