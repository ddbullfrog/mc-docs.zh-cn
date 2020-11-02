---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: Show Columns - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW COLUMNS 语法。
ms.openlocfilehash: ff17b0f4baf18c9d265318be44058104746e4b59
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472850"
---
# <a name="show-columns"></a>显示列

```sql
SHOW COLUMNS (FROM | IN) [db_name.]table_name
```

返回表中的列的列表。 如果该表不存在，则会引发异常。