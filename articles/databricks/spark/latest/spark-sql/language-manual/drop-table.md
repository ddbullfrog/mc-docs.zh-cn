---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 删除表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 DROP TABLE 语法。
ms.openlocfilehash: 4322a3d17ec1d79bb84c87ce06c83aff82156a1c
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472996"
---
# <a name="drop-table"></a>删除表

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name
```

如果表不是 `EXTERNAL` 表，请删除该表并从文件系统中删除与该表关联的目录。 如果要删除的表不存在，则会引发异常。

**`IF EXISTS`**

如果该表不存在，则不会执行任何操作。