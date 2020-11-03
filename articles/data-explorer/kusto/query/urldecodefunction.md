---
title: url_decode() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 url_decode()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 7e602017ec2ec51e752c954ccb845da81c4b71db
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105964"
---
# <a name="url_decode"></a>url_decode()

此函数将编码的 URL 转换为常规 URL 表示形式。 

## <a name="syntax"></a>语法

`url_decode(`encoded url`)`

## <a name="arguments"></a>参数

* encoded url：编码的 URL（字符串）。  

## <a name="returns"></a>返回

常规表示形式的 URL（字符串）。

## <a name="examples"></a>示例

```kusto
let url = @'https%3a%2f%2fwww.bing.com%2f';
print original = url, decoded = url_decode(url)
```

|原配|已解码|
|---|---|
|https%3a%2f%2f www.bing.com%2f|https://www.bing.com/|



 