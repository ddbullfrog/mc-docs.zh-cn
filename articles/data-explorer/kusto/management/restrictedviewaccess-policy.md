---
title: Kusto RestrictedViewAccess 策略管理 - Azure 数据资源管理器
description: 了解 Azure 数据资源管理器中的 RestrictedViewAccess 策略命令。 了解如何查看、启用、禁用、更改和删除此策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: 703c909aa54ac3247d629f4b812370f3a56709a8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106341"
---
# <a name="restricted_view_access-policy-command"></a>restricted_view_access 策略命令

RestrictedViewAccess 策略的信息可参见[此处](../management/restrictedviewaccesspolicy.md)。

用于在数据库中的表上启用或禁用策略的控制命令如下所示：

启用/禁用策略：
```kusto
.alter table TableName policy restricted_view_access true|false
```

启用/禁用多个表的策略：
```kusto
.alter tables (TableName1, ..., TableNameN) policy restricted_view_access true|false
```

查看策略：
```kusto
.show table TableName policy restricted_view_access  

.show table * policy restricted_view_access  
```

删除策略（与禁用行为等效）：
```kusto
.delete table TableName policy restricted_view_access  
```
