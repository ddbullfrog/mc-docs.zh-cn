---
title: parse_ipv4_mask() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 parse_ipv4_mask() 函数。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 05/27/2020
ms.date: 10/29/2020
ms.openlocfilehash: 26ae8ca289f5cad31b5d849959107f3a57a5ea13
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105216"
---
# <a name="parse_ipv4_mask"></a>parse_ipv4_mask()

将 IPv4 和网络掩码的输入字符串转换为长数字表示形式（带符号的 64 位）。

```kusto
parse_ipv4_mask("127.0.0.1", 24) == 2130706432
parse_ipv4_mask('192.1.168.2', 31) == parse_ipv4_mask('192.1.168.3', 31)
```

## <a name="syntax"></a>语法

`parse_ipv4_mask(`*`Expr`*`, `*`PrefixMask`*`)`

## <a name="arguments"></a>参数

* *`Expr`* ：将转换为 long 类型的 IPv4 地址的字符串表示形式。 
* *`PrefixMask`* ：从 0 到 32 的整数，表示所考虑的最有效位的数目。

## <a name="returns"></a>返回

如果转换成功，则结果将是一个 long 类型的数字。
如果转换未成功，结果将为 `null`。
