---
title: Kusto IngestionTime 策略管理 - Azure 数据资源管理器
description: 熟悉 Azure 数据资源管理器中的 IngestionTime 策略命令。 了解如何访问引入时间，并了解如何启用和禁用此策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: ba2d70682ea92676c7e611ab7998238a9d75fa8b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106263"
---
# <a name="ingestiontime-policy-command"></a>ingestiontime 策略命令

IngestionTime 策略是针对表设置的可选策略（默认启用）。
它提供将记录引入表的大致时间。

可以使用 `ingestion_time()` 函数在查询时访问引入时间值。

```kusto
T | extend ingestionTime = ingestion_time()
```

启用/禁用策略：
```kusto
.alter table table_name policy ingestiontime true
```

启用/禁用多个表的策略：
```kusto
.alter tables (table_name [, ...]) policy ingestiontime true
```

查看策略：
```kusto
.show table table_name policy ingestiontime  

.show table * policy ingestiontime  
```

删除策略（与禁用行为等效）：
```kusto
.delete table table_name policy ingestiontime  
```
