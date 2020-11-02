---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: Describe Database - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 DESCRIBE DATABASE 语法。
ms.openlocfilehash: f4c121a3ebbab75608ec802d3ecff0c2f168f2c4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472813"
---
# <a name="describe-database"></a>描述数据库

```sql
DESCRIBE DATABASE [EXTENDED] db_name
```

返回现有数据库的元数据（名称、注释和位置）。 如果该数据库不存在，则会引发异常。

**`EXTENDED`**

显示数据库属性。