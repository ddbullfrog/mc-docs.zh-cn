---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 显示表属性 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW TBLPROPERTIES 语法。
ms.openlocfilehash: b49248973a4a2e9177a8037d29f412f8e5a59e1b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472983"
---
# <a name="show-table-properties"></a>显示表属性

```sql
SHOW TBLPROPERTIES [db_name.]table_name [property_key]
```

返回表中的所有属性或特定属性集的值。 如果该表不存在，会引发异常。