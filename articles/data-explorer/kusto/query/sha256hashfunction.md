---
title: Hash_sha256() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 hash_sha256()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: b7ebb05aca378356aeefa521cfbe58779f9bd952
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106027"
---
# <a name="hash_sha256"></a>hash_sha256()

返回输入值的 sha256 哈希值。

## <a name="syntax"></a>语法

`hash_sha256(`*source*`)`

## <a name="arguments"></a>参数

* *source* ：要进行哈希处理的值。

## <a name="returns"></a>返回

给定标量的 sha256 哈希值编码为十六进制字符串（其中的每两个字符都表示 0 到 255 之间的一个十六进制数字）。

> [!WARNING]
> 此函数 (SHA256) 所用的算法可确保不会在以后修改，但计算起来非常复杂。 建议需要在单个查询期间使用“轻量级”哈希函数的用户改为使用函数 [hash()](./hashfunction.md)。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print 
h1=hash_sha256("World"),
h2=hash_sha256(datetime(2020-01-01))
```

|h1|h2|
|---|---|
|78ae647dc5544d227130a0682a51e30bc7777fbb6d8a8f17007463a3ecd1d524|ba666752dc1a20eb750b0eb64e780cc4c968bc9fb8813461c1d7e750f302d71d|

以下示例使用 `hash_sha256()` 函数根据 State 的 SHA256 哈希值聚合 StormEvents。 

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents 
| summarize StormCount = count() by State, StateHash=hash_sha256(State)
| top 5 by StormCount desc
```

|状态|StateHash|StormCount|
|---|---|---|
|德克萨斯|9087f20f23f91b5a77e8406846117049029e6798ebbd0d38aea68da73a00ca37|4701|
|堪萨斯州|c80e328393541a3181b258cdb4da4d00587c5045e8cf3bb6c8fdb7016b69cc2e|3166|
|衣阿华州|f85893dca466f779410f65cd904fdc4622de49e119ad4e7c7e4a291ceed1820b|2337|
|伊利诺斯州|ae3eeabfd7eba3d9a4ccbfed6a9b8cff269dc43255906476282e0184cf81b7fd|2022|
|密苏里州|d15dfc28abc3ee73b7d1f664a35980167ca96f6f90e034db2a6525c0b8ba61b1|2016|
