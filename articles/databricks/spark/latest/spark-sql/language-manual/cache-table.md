---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 缓存表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 CACHE TABLE 语法。
ms.openlocfilehash: 5e69112bc4c61db90084687d1f18efcf9f6804da
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472752"
---
# <a name="cache-table"></a>缓存表

```sql
CACHE [LAZY] TABLE [db_name.]table_name
```

使用 RDD 缓存将表的内容缓存在内存中。 这使得后续查询可以尽可能避免扫描原始文件。

**`LAZY`**

延迟缓存该表，而不是急于扫描整个表。

请参阅[增量和 Apache Spark 缓存](../../../../delta/optimizations/delta-cache.md#delta-and-rdd-cache-comparison)以了解 RDD 缓存和 Databricks IO 缓存之间的差异。