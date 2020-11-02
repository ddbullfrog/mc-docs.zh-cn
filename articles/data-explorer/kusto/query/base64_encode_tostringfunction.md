---
title: base64_encode_tostring() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 base64_encode_tostring()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 06/22/2019
ms.date: 10/29/2020
ms.openlocfilehash: 3e38b7b2cf4a1c043b2873609bd3451c92bc196a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104042"
---
# <a name="base64_encode_tostring"></a>base64_encode_tostring()

将字符串编码为 base64 字符串。

## <a name="syntax"></a>语法

`base64_encode_tostring(`String`)`

## <a name="arguments"></a>参数

* String：要编码为 base64 字符串的输入字符串。

## <a name="returns"></a>返回

返回编码为 base64 字符串的字符串。

* 若要将 base64 字符串解码为 UTF-8 字符串，请参阅 [base64_decode_tostring()](base64_decode_tostringfunction.md)
* 若要将 base64 字符串解码为长值数组，请参阅 [base64_decode_toarray()](base64_decode_toarrayfunction.md)


## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print Quine=base64_encode_tostring("Kusto")
```

|Quine   |
|--------|
|S3VzdG8=|

