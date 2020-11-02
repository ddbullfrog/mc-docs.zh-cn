---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/03/2020
title: 删除视图 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 DROP VIEW 语法。
ms.openlocfilehash: e04ca6d0a24b5ef83c1bf03afe5c6be20ef622f0
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472993"
---
# <a name="drop-view"></a>删除视图

```sql
DROP VIEW [db_name.]view_name
```

删除基于一个或多个表的逻辑视图。

## <a name="examples"></a>示例

```sql
-- Drop the global temp view, temp view, and persistent view.
DROP VIEW global_temp.global_DeptSJC;
DROP VIEW temp_DeptSFO;
DROP VIEW database1.view_deptDetails;
```