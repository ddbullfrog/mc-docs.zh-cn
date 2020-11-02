---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 显示表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW TABLES 语法。
ms.openlocfilehash: abb1a7505728f6db2c4792201063c2fd879dfa4b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473130"
---
# <a name="show-tables"></a>显示表

```sql
SHOW TABLES [FROM | IN] db_name [LIKE 'pattern']
```

返回所有表。 显示表的数据库以及表是否是临时表。

**`FROM | IN`**

返回数据库中的所有表。

**`LIKE 'pattern'`**

指示要匹配的表名称。 在 `pattern` 中，`*` 匹配任意数量的字符。