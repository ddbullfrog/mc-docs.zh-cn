---
title: zlib_compress_to_base64_string - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 zlib_compress_to_base64_string() 命令。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: elgevork
ms.service: data-explorer
ms.topic: reference
origin.date: 09/29/2020
ms.date: 10/30/2020
ms.openlocfilehash: 4835357c6360d408e7fa67d7941243b7f708e601
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106556"
---
# <a name="zlib_compress_to_base64_string"></a>zlib_compress_to_base64_string()

执行 zlib 压缩，并将结果编码为 base64。

> [!NOTE]
> 唯一受支持的 Windows 大小为 15。

## <a name="syntax"></a>语法

`zlib_compress_to_base64_string('input_string')`

## <a name="arguments"></a>参数

input_string：输入 `string`（要压缩并进行 base64 编码的字符串）。 此函数接受一个字符串参数。

## <a name="returns"></a>返回

* 返回 `string`，它表示已进行 zlib 压缩和 base64 编码的原始字符串。 
* 如果压缩或编码失败，则返回空结果。

## <a name="example"></a>示例

### <a name="using-kusto"></a>使用 Kusto

```kusto
print zcomp = zlib_compress_to_base64_string("1234567890qwertyuiop")
```

**输出：** |"eAEBFADr/zEyMzQ1Njc4OTBxd2VydHl1aW9wOAkGdw=="|

### <a name="using-python"></a>使用 Python

可以使用其他工具（例如 Python）完成压缩： 

```python
print(base64.b64encode(zlib.compress(b'<original_string>')))
```

## <a name="next-steps"></a>后续步骤

使用 [zlib_decompress_from_base64_string()](zlib-base64-decompress.md) 检索未压缩的原始字符串。