---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 删除数据库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 DROP DATABASE 和 DROP SCHEMA 语法。
ms.openlocfilehash: 1ece3b38907177259e839a81f9ca6bfbb4e4ddff
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472999"
---
# <a name="drop-database"></a>删除数据库

```sql
DROP [DATABASE | SCHEMA] [IF EXISTS] db_name [RESTRICT | CASCADE]
```

删除数据库并从文件系统中删除与该数据库关联的目录。 如果该数据库不存在，则会引发异常。

**`IF EXISTS`**

如果要删除的数据库不存在，则什么都不会发生。

**`RESTRICT`**

删除非空数据库会触发异常。 默认情况下启用。

**`CASCADE`**

删除非空数据库也会导致所有关联的表和函数被删除。