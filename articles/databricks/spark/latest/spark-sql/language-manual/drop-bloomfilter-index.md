---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: Drop Bloom Filter Index - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 DROP BLOOMFILTER INDEX 语法。
ms.openlocfilehash: 56cfb8d9ff15464786fcd61b2fe00111fb3f6b67
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473000"
---
# <a name="drop-bloom-filter-index"></a>删除 Bloom 筛选器索引

```sql
DROP BLOOMFILTER INDEX
ON [TABLE] table_name
[FOR COLUMNS(columnName1, columnName2, ...)]
```

删除 Bloom 筛选器索引。

如果表名或其中一列不存在，该命令就会失败。 所有与 Bloom 筛选器相关的元数据都将从指定列中删除。

如果表中没有任何 Bloom 筛选器，则在清理表时将清除基础索引文件。