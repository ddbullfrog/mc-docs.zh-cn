---
title: parse_ipv6() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 parse_ipv6() 函数。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 05/27/2020
ms.date: 10/29/2020
ms.openlocfilehash: f80e957963ab4d226e292649de9491d6074d0639
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103835"
---
# <a name="parse_ipv6"></a>parse_ipv6()

将 IPv6 或 IPv4 字符串转换为规范 IPv6 字符串表示形式。

```kusto
parse_ipv6("127.0.0.1") == '0000:0000:0000:0000:0000:ffff:7f00:0001'
parse_ipv6(":fe80::85d:e82c:9446:7994") == 'fe80:0000:0000:0000:085d:e82c:9446:7994'
```

## <a name="syntax"></a>语法

`parse_ipv6(`*`Expr`*`)`

## <a name="arguments"></a>参数

* *`Expr`* ：表示将转换为规范 IPv6 表示形式的 IPv6/IPv4 网络地址的字符串表达式。 字符串可以包含使用 [IP 前缀表示法](#ip-prefix-notation)的网络掩码。

## <a name="ip-prefix-notation"></a>IP 前缀表示法

IP 地址可通过使用左斜线 (`/`) 字符的 `IP-prefix notation` 进行定义。
左斜线 (`/`) 左边的 IP 地址是基本 IP 地址。 左斜线 (`/`) 右边的数字（1 到 127）是网络掩码中连续 1 位的数目。

## <a name="returns"></a>返回

如果转换成功，则结果将是表示规范 IPv6 网络地址的字符串。
如果转换未成功，结果将为 `null`。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(ip_string:string, netmask:long)
[
 '192.168.255.255',     32,  // 32-bit netmask is used
 '192.168.255.255/24',  30,  // 24-bit netmask is used, as IPv4 address doesn't use upper 8 bits
 '255.255.255.255',     24,  // 24-bit netmask is used
]
| extend ip_long = parse_ipv4_mask(ip_string, netmask)
```

|ip_string|网络掩码|ip_long|
|---|---|---|
|192.168.255.255|32|3232301055|
|192.168.255.255/24|30|3232300800|
|255.255.255.255|24|4294967040|


