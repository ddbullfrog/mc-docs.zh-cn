---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 加载数据 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 LOAD DATA 语法。
ms.openlocfilehash: 854968e36b37d4b43176b1bb754a3b43b0453c4b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473016"
---
# <a name="load-data"></a>加载数据

```sql
LOAD DATA [LOCAL] INPATH path [OVERWRITE] INTO TABLE [db_name.]table_name [PARTITION part_spec]

part_spec:
    : (part_col_name1=val1, part_col_name2=val2, ...)
```

将数据从文件加载到表或表中的分区中。 目标表不能是临时表。 当且仅当目标表已分区时，才必须提供分区规范。

> [!NOTE]
>
> 仅使用 Hive 格式创建的表支持此功能。

**`LOCAL`**

从本地文件系统加载路径。 否则，将使用默认文件系统。

**`OVERWRITE`**

删除表中的现有数据。 否则，新数据将追加到该表中。