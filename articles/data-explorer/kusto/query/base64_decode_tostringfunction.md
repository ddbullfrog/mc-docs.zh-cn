---
title: base64_decode_tostring() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 base64_decode_tostring()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 06/22/2019
ms.date: 10/29/2020
ms.openlocfilehash: 72b29be9c4c5b62b07826d634cb26b9dc0474887
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104051"
---
# <a name="base64_decode_tostring"></a>base64_decode_tostring()

将 base64 字符串解码为 UTF-8 字符串。

## <a name="syntax"></a>语法

`base64_decode_tostring(`*String*`)`

## <a name="arguments"></a>参数

* *String* ：输入要从 base64 解码为 UTF-8 字符串的字符串。

## <a name="returns"></a>返回

返回从 base64 字符串解码的 UTF-8 字符串。

* 若要将 base64 字符串解码为长值数组，请参阅 [base64_decode_toarray()](base64_decode_toarrayfunction.md)
* 若要将字符串解码为 base64 字符串，请参阅 [base64_encode_tostring()](base64_encode_tostringfunction.md)

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print Quine=base64_decode_tostring("S3VzdG8=")
```

|Quine|
|-----|
|Kusto|

如果尝试对通过无效 UTF-8 编码生成的 base64 字符串进行解码，将返回 null：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print Empty=base64_decode_tostring("U3RyaW5n0KHR0tGA0L7Rh9C60LA=")
```

|空|
|-----|
||
