---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 显示数据库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW DATABASES 和 SHOW SCHEMAS 语法。
ms.openlocfilehash: 2182abb283368368fdb0c31040227947d0b26177
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472848"
---
# <a name="show-databases"></a>显示数据库

```sql
SHOW [DATABASES | SCHEMAS] [LIKE 'pattern']
```

返回所有数据库。 `SHOW SCHEMAS` 是 `SHOW DATABASES`的同义词。

**`LIKE 'pattern'`**

要匹配的数据库名称。 在 `pattern` 中，`*` 匹配任意数量的字符。