---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/03/2020
title: 更改数据库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 ALTER DATABASE 语法。
ms.openlocfilehash: e65dbe4d03469f11185bd38481ef537e829fc20c
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473041"
---
# <a name="alter-database"></a>更改数据库

```sql
ALTER [DATABASE|SCHEMA] db_name SET DBPROPERTIES (key=val, ...)
```

**`SET DBPROPERTIES`**

为数据库指定名为 `key` 的属性，并将该属性的值设置为 `val`。 如果 `key` 已存在，则 `val` 将覆盖旧值。

## <a name="assign-owner"></a>分配所有者

```sql
ALTER DATABASE db_name OWNER TO `user_name@user_domain.com`
```

为数据库分配所有者。