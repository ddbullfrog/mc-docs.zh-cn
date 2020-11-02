---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 显示授权 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW GRANT 语法。
ms.openlocfilehash: d797180ff5f9120f2718a17ed9377cdbb4b0a6b5
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472748"
---
# <a name="show-grant"></a>显示授权

```sql
SHOW GRANT [user] ON [CATALOG | DATABASE <database-name> | TABLE <table-name> | VIEW <view-name> | FUNCTION <function-name> | ANONYMOUS FUNCTION | ANY FILE]
```

显示会影响指定对象的所有权限（包括已继承、已拒绝和已授予的）。

**示例**

```sql
SHOW GRANT `<user>@<domain-name>` ON DATABASE <database-name>
```