---
title: .clear table data - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 `.clear table data` 命令。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: vrozov
ms.service: data-explorer
ms.topic: reference
origin.date: 10/01/2020
ms.date: 10/30/2020
ms.openlocfilehash: 68b95230eba370e35feba384e84ce9c16df459af
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106761"
---
# <a name="clear-table-data"></a>.clear table data

清除现有表的数据，包括流式处理引入数据。

`.clear` `table` *TableName* `data` 

> [!NOTE]
> * 需要[表管理员权限](../management/access-control/role-based-authorization.md)

**示例** 

```kusto
.clear table LyricsAsTable data 
```
 
