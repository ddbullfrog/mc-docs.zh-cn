---
title: hash() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 hash()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 0d0e6c452fc4afde3ae89fd424b4d9a8575b9788
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103878"
---
# <a name="hash"></a>hash()

返回输入值的哈希值。

## <a name="syntax"></a>语法

`hash(`*source* [`,` *mod* ]`)`

## <a name="arguments"></a>参数

* *source* ：要进行哈希处理的值。
* mod：一个可选模块值，在应用于哈希结果后输出值将在 `0` 到 mod - 1 之间

## <a name="returns"></a>返回

给定标量的哈希值，对给定 mod 值（如果已指定）取模。

> [!WARNING]
> 用于计算哈希的算法是 xxhash。
> 此算法将来可能会更改，唯一的保证是：在单个查询中，对这个方法的所有调用都使用相同的算法。
> 因此，建议不要将 `hash()` 的结果存储在表中。 如果需要持久化哈希值，请改用 [hash_sha256()](./sha256hashfunction.md) 或 [hash_md5()](./md5hashfunction.md)。 请注意，计算这些函数比 `hash()` 更复杂。

## <a name="examples"></a>示例

```kusto
hash("World")                   // 1846988464401551951
hash("World", 100)              // 51 (1846988464401551951 % 100)
hash(datetime("2015-01-01"))    // 1380966698541616202
```

下面的示例使用哈希函数对 10% 的数据运行查询。假设值是均匀分布的（在本示例中，值为 StartTime 值），则使用哈希函数对数据采样很有帮助。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where hash(StartTime, 10) == 0
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State 
| top 5 by StormCount desc
```
