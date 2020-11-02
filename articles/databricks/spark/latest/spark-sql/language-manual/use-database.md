---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 使用数据库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 USE (database) 语法。
ms.openlocfilehash: 7b25826c6f3574bcc07a7b7988c6878fbd1e6b10
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472834"
---
# <a name="use-database"></a>使用数据库

```sql
USE db_name
```

设置当前数据库。 未显式指定数据库的所有后续命令将使用此数据库。 如果提供的数据库不存在，则会引发异常。 默认的当前数据库是 `default`。