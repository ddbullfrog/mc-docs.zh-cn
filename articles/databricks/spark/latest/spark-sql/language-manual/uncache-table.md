---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: Uncache Table - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 UNCACHE TABLE 语法。
ms.openlocfilehash: 49d223f6bc78e57d75a81149ab20b188229b0390
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472829"
---
# <a name="uncache-table"></a>取消缓存表

```sql
UNCACHE TABLE [db_name.]table_name
```

从 RDD 缓存中删除与该表关联的所有缓存条目。