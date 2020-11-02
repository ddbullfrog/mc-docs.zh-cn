---
title: pack() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 pack()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 49682e3891c889dc87e1cb8a90e062804951eaa2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103843"
---
# <a name="pack"></a>pack()

从名称和值的列表创建一个 `dynamic` 对象（属性包）。

`pack_dictionary()` 函数的别名。

## <a name="syntax"></a>语法

`pack(`*key1*`,` *value1*`,` *key2*`,` *value2*`,... )`

## <a name="arguments"></a>参数

* 键和值的交替列表（列表的总长度必须为偶数）
* 所有键都必须为非空常量字符串

## <a name="examples"></a>示例

以下示例返回 `{"Level":"Information","ProcessID":1234,"Data":{"url":"www.bing.com"}}`：

```kusto
pack("Level", "Information", "ProcessID", 1234, "Data", pack("url", "www.bing.com"))
```

让我们看看 2 个表：SmsMessages 和 MmsMessages：

表 SmsMessages 

|SourceNumber |TargetNumber| CharsCount
|---|---|---
|555-555-1234 |555-555-1212 | 46 
|555-555-1234 |555-555-1213 | 50 
|555-555-1212 |555-555-1234 | 32 

表 MmsMessages 

|SourceNumber |TargetNumber| AttachmentSize | AttachmentType | AttachmentName
|---|---|---|---|---
|555-555-1212 |555-555-1213 | 200 | jpeg | Pic1
|555-555-1234 |555-555-1212 | 250 | jpeg | Pic2
|555-555-1234 |555-555-1213 | 300 | png | Pic3

以下查询：
```kusto
SmsMessages 
| extend Packed=pack("CharsCount", CharsCount) 
| union withsource=TableName kind=inner 
( MmsMessages 
  | extend Packed=pack("AttachmentSize", AttachmentSize, "AttachmentType", AttachmentType, "AttachmentName", AttachmentName))
| where SourceNumber == "555-555-1234"
``` 

返回：

|TableName |SourceNumber |TargetNumber | Packed
|---|---|---|---
|SmsMessages|555-555-1234 |555-555-1212 | {"CharsCount":46}
|SmsMessages|555-555-1234 |555-555-1213 | {"CharsCount":50}
|MmsMessages|555-555-1234 |555-555-1212 | {"AttachmentSize":250, "AttachmentType": "jpeg", "AttachmentName":"Pic2"}
|MmsMessages|555-555-1234 |555-555-1213 | {"AttachmentSize":300, "AttachmentType": "png", "AttachmentName":"Pic3"}
