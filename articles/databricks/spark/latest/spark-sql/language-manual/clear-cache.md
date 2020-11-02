---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 清除缓存 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 CLEAR CACHE 语法。
ms.openlocfilehash: 8350eff274cba24d97511defafd43fe1d9b0b807
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472736"
---
# <a name="clear-cache"></a>清除缓存

```sql
CLEAR CACHE
```

清除与 SQLContext 关联的 RDD 缓存。