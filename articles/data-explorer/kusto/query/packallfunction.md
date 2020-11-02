---
title: pack_all() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 pack_all()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: b34273ac8b01e722c9e4a76464e5278d0539b6af
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103847"
---
# <a name="pack_all"></a>pack_all()

从表格表达式的所有列创建一个 `dynamic` 对象（属性包）。

> [!NOTE]
> 不保证返回对象的表示形式在运行之间为字节级兼容。 例如，包中出现的属性可能以不同的顺序出现。

## <a name="syntax"></a>语法

`pack_all()`

## <a name="examples"></a>示例

给定表 SmsMessages 

|SourceNumber |TargetNumber| CharsCount
|---|---|---
|555-555-1234 |555-555-1212 | 46 
|555-555-1234 |555-555-1213 | 50 
|555-555-1212 |555-555-1234 | 32 

以下查询：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(SourceNumber:string,TargetNumber:string,CharsCount:long)
[
'555-555-1234','555-555-1212',46,
'555-555-1234','555-555-1213',50,
'555-555-1212','555-555-1234',32
]
| extend Packed=pack_all()
```

返回：

|TableName |SourceNumber |TargetNumber | Packed
|---|---|---|---
|SmsMessages|555-555-1234 |555-555-1212 | {"SourceNumber":"555-555-1234", "TargetNumber":"555-555-1212", "CharsCount":46%
|SmsMessages|555-555-1234 |555-555-1213 | {"SourceNumber":"555-555-1234", "TargetNumber":"555-555-1213", "CharsCount":50}
|SmsMessages|555-555-1212 |555-555-1234 | {"SourceNumber":"555-555-1212", "TargetNumber":"555-555-1234", "CharsCount":32}
