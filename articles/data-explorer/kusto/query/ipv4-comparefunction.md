---
title: ipv4_compare() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 ipv4_compare()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: 68c2e1230685f5459df8abd876997502afd7ba23
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106008"
---
# <a name="ipv4_compare"></a>ipv4_compare()

比较两个 IPv4 字符串。 分析并比较两个 IPv4 字符串，同时考虑根据参数前缀和可选的 `PrefixMask` 参数计算出的组合 IP 前缀掩码。

```kusto
ipv4_compare("127.0.0.1", "127.0.0.1") == 0
ipv4_compare('192.168.1.1', '192.168.1.255') < 0
ipv4_compare('192.168.1.1/24', '192.168.1.255/24') == 0
ipv4_compare('192.168.1.1', '192.168.1.255', 24) == 0
```

## <a name="syntax"></a>语法

`ipv4_compare(`*Expr1*`, `*Expr2*`[ ,`*PrefixMask*`])`

## <a name="arguments"></a>参数

* Expr1、Expr2：表示 IPv4 地址的字符串表达式。 可以使用 [IP 前缀表示法](#ip-prefix-notation)对 IPv4 字符串进行掩码操作。
* PrefixMask：从 0 到 32 的整数，表示所考虑的最有效位的数目。

## <a name="ip-prefix-notation"></a>IP 前缀表示法
 
IP 地址可以通过斜杠 (`/`) 字符使用 `IP-prefix notation` 进行定义。
斜杠 (`/`) 左边的 IP 地址是基本 IP 地址。 斜杠 (`/`) 右边的数字（1 到 32）是网络掩码中连续位的数目。 

例如，192.168.2.0/24 将具有关联的网络/子网掩码，其中包含 24 个连续位或点分十进制格式的 255.255.255.0。

## <a name="returns"></a>返回

* `0`：如果第一个 IPv4 字符串参数的长表示形式等于第二个 IPv4 字符串参数
* `1`：如果第一个 IPv4 字符串参数的长表示形式大于第二个 IPv4 字符串参数
* `-1`：如果第一个 IPv4 字符串参数的长表示形式小于第二个 IPv4 字符串参数
* `null`：如果两个 IPv4 字符串之一转换不成功。

## <a name="examples-ipv4-comparison-equality-cases"></a>示例:IPv4 比较相等的情况

### <a name="compare-ips-using-the-ip-prefix-notation-specified-inside-the-ipv4-strings"></a>使用 IPv4 字符串中指定的 IP 前缀表示法比较 IP

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string)
[
 '192.168.1.0',    '192.168.1.0',       // Equal IPs
 '192.168.1.1/24', '192.168.1.255',     // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255/24',  // 24 bit IP-prefix is used for comparison
 '192.168.1.1/30', '192.168.1.255/24',  // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_compare(ip1_string, ip2_string)
```

|ip1_string|ip2_string|result|
|---|---|---|
|192.168.1.0|192.168.1.0|0|
|192.168.1.1/24|192.168.1.255|0|
|192.168.1.1|192.168.1.255/24|0|
|192.168.1.1/30|192.168.1.255/24|0|

### <a name="compare-ips-using-ip-prefix-notation-specified-inside-the-ipv4-strings-and-as-additional-argument-of-the-ipv4_compare-function"></a>使用 IPv4 字符串中指定的 IP 前缀表示法作为 `ipv4_compare()` 函数的附加参数比较 IP

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string, prefix:long)
[
 '192.168.1.1',    '192.168.1.0',   31, // 31 bit IP-prefix is used for comparison
 '192.168.1.1/24', '192.168.1.255', 31, // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255', 24, // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_compare(ip1_string, ip2_string, prefix)
```

|ip1_string|ip2_string|前缀|result|
|---|---|---|---|
|192.168.1.1|192.168.1.0|31|0|
|192.168.1.1/24|192.168.1.255|31|0|
|192.168.1.1|192.168.1.255|24|0|

