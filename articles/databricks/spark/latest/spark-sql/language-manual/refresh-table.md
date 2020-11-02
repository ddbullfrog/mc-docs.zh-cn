---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 刷新表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 REFRESH TABLE 语法。
ms.openlocfilehash: dea671658d2a5122c46fc872fce6eaa5e6c76a3d
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473011"
---
# <a name="refresh-table"></a>刷新表

```sql
REFRESH TABLE [db_name.]table_name
```

刷新与该表关联的所有缓存条目。 如果以前缓存过该表，则会在下次扫描时延迟缓存该表。