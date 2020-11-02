---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/30/2020
title: 创建数据库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 CREATE DATABASE 和 CREATE SCHEMA 语法。
ms.openlocfilehash: 243de5b6427bc61903afdc3c357515b3c7365111
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472820"
---
# <a name="create-database"></a>创建数据库

```sql
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] db_name
  [COMMENT comment_text]
  [LOCATION path]
  [WITH DBPROPERTIES (key=val, ...)]
```

创建数据库。 如果已存在同名数据库，则会引发异常。

**`IF NOT EXISTS`**

如果已存在同名数据库，则不会执行任何操作。

**`LOCATION`**

如果基础文件系统中不存在指定的路径，此命令将尝试使用路径创建目录。

**`WITH DBPROPERTIES`**

为数据库指定名为 `key` 的属性，并将该属性的值设置为 `val`。 如果 `key` 已存在，则 `val` 将覆盖旧值。

## <a name="examples"></a>示例

```sql
-- Create database `customer_db`. This throws exception if database with name customer_db
-- already exists.
CREATE DATABASE customer_db;

-- Create database `customer_db` only if database with same name doesn't exist.
CREATE DATABASE IF NOT EXISTS customer_db;
```