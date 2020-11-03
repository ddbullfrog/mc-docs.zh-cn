---
title: parse_ipv4() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 parse_ipv4()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: 310f5bfe8fa0502dda8ed57b944606ff9981f944
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106029"
---
# <a name="parse_ipv4"></a>parse_ipv4()

将 IPv4 字符串转换为 long（带符号的 64 位）数字表示形式。

```kusto
parse_ipv4("127.0.0.1") == 2130706433
parse_ipv4('192.1.168.1') < parse_ipv4('192.1.168.2') == true
```

## <a name="syntax"></a>语法

`parse_ipv4(`*`Expr`*`)`

## <a name="arguments"></a>参数

* *`Expr`* ：字符串表达式，表示将转换为 long 的 IPv4。 字符串可以包含使用 [IP 前缀表示法](#ip-prefix-notation)的网络掩码。

## <a name="ip-prefix-notation"></a>IP 前缀表示法

IP 地址可通过使用斜杠 (`/`) 字符的 `IP-prefix notation` 进行定义。
斜杠 (`/`) 左边的 IP 地址是基本 IP 地址。 斜杠 (/) 右边的数字（1 到 32）是网络掩码中连续位的数目。

例如，192.168.2.0/24 将具有关联的网络/子网掩码，其中包含 24 个连续位或点分十进制格式的 255.255.255.0。

## <a name="returns"></a>返回

如果转换成功，则结果将是一个 long 类型的数字。
如果转换未成功，结果将为 `null`。
 
## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(ip_string:string)
[
 '192.168.1.1',
 '192.168.1.1/24',
 '255.255.255.255/31'
]
| extend ip_long = parse_ipv4(ip_string)
```

|ip_string|ip_long|
|---|---|
|192.168.1.1|3232235777|
|192.168.1.1/24|3232235776|
|255.255.255.255/31|4294967294|
