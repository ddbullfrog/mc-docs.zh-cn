---
title: zlib_decompress_from_base64_string() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 zlib_decompress_from_base64_string() 命令。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: elgevork
ms.service: data-explorer
ms.topic: reference
origin.date: 09/29/2020
ms.date: 10/30/2020
ms.openlocfilehash: f8cb2912943304c22ee5dc3e7d2fa1b01d498972
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106555"
---
# <a name="zlib_decompress_from_base64_string"></a>zlib_decompress_from_base64_string()

从 base64 对输入字符串进行解码，并执行 zlib 解压缩。

> [!NOTE]
> 唯一受支持的 Windows 大小为 15。

## <a name="syntax"></a>语法

`zlib_decompress_from_base64_string('input_string')`

## <a name="arguments"></a>参数

input_string：输入 `string`，该项已用 zlib 压缩并进行了 base64 编码。 此函数接受一个字符串参数。

## <a name="returns"></a>返回

* 返回一个表示原始字符串的 `string`。 
* 如果解压缩或解码失败，则返回空结果。 
    * 例如，无效的经过 zlib 压缩和 base64 编码的字符串会返回空输出。

## <a name="examples"></a>示例

```kusto
print zcomp = zlib_decompress_from_base64_string("eJwLSS0uUSguKcrMS1cwNDIGACxqBQ4=")
```

**输出：**

|Test string 123|

无效输入的示例：

```kusto
print zcomp = zlib_decompress_from_base64_string("x0x0x0")
```

**输出：** 
||

## <a name="next-steps"></a>后续步骤

使用 [zlib_compress_to_base64_string()](zlib-base64-compress.md) 创建压缩的输入字符串。