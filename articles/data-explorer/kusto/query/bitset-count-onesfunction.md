---
title: bitset_count_ones() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 bitset_count_ones()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/22/2020
ms.date: 10/29/2020
ms.openlocfilehash: 788bfd31f6011659a93090c514e5119102e9b05c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106045"
---
# <a name="bitset_count_ones"></a>bitset_count_ones()

返回数字的二进制表示形式中的设置位数。

```kusto
bitset_count_ones(42)
```

## <a name="syntax"></a>语法

`bitset_count_ones(`*num1* ``)`

## <a name="arguments"></a>参数

* *num1* ：长整型或整数。

## <a name="returns"></a>返回

返回数字的二进制表示形式中的设置位数。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
// 42 = 32+8+2 : b'00101010' == 3 bits set
print ones = bitset_count_ones(42) 
```

|个|
|---|
|3|
