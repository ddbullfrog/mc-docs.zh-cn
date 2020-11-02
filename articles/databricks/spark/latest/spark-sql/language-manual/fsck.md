---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: Fsck Repair Table（Azure Databricks 上的 Delta Lake） - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 FSCK REPAIR TABLE 语法。
ms.openlocfilehash: deb535a8b64e64428208badf0fc4cf7ddf464755
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472985"
---
# <a name="fsck-repair-table--delta-lake-on-azure-databricks"></a>对表进行 Fsck 修复（Azure Databricks 上的 Delta Lake）

```sql
FSCK REPAIR TABLE [db_name.]table_name [DRY RUN]
```

从 Delta 表的事务日志中删除无法再从基础文件系统中找到的文件条目。 手动删除这些文件时，可能会发生这种情况。

**`DRY RUN`**

返回要从事务日志中删除的文件的列表。