---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 显示用于创建表的命令 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW CREATE TABLE 语法。
ms.openlocfilehash: 678b393d802530d78e8436d1f5e82e92f93fa910
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472849"
---
# <a name="show-create-table"></a>显示用于创建表的命令

```sql
SHOW CREATE TABLE [db_name.]table_name
```

返回用于创建现有表的命令。 如果该表不存在，则会引发异常。