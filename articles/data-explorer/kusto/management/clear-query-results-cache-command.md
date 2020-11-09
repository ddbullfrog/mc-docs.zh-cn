---
title: 清除查询结果缓存 - Azure 数据资源管理器
description: 了解如何在 Azure 数据资源管理器中清除缓存的查询结果。 了解要使用的命令并查看示例。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: amitof
ms.service: data-explorer
ms.topic: reference
origin.date: 06/16/2020
ms.date: 10/29/2020
ms.openlocfilehash: b5d4d0df26eed8dd65216399c01aa0808ce55619
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106119"
---
# <a name="clear-query-results-cache"></a>清除查询结果缓存

清除针对上下文数据库所获得的所有[缓存的查询结果](../query/query-results-cache.md)。

**语法**

`.clear` `database` `cache` `query_results`

**返回**

此命令返回包含以下列的表：

|列    |类型    |说明
|---|---|---
|NodeId|`string`|群集节点的标识符。
|计数|`long`|节点删除的项数。

**示例**

```kusto
.clear database cache queryresults
```

|NodeId|项|
|---|---|
|Node1|42
|Node2|0
