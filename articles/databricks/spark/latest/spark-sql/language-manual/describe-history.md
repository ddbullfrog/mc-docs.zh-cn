---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 描述历史记录（Azure Databricks 上的 Delta Lake）- Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 DESCRIBE TABLE 语法。
ms.openlocfilehash: 85e84bbade228db402cdfefdabd97ddf14e3de70
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473117"
---
# <a name="describe-history-delta-lake-on-azure-databricks"></a><a id="describe-history"> </a><a id="describe-history-delta-lake-on-azure-databricks"> </a>描述历史记录（Azure Databricks 上的 Delta Lake）

```sql
DESCRIBE HISTORY [db_name.]table_name

DESCRIBE HISTORY delta.`<path-to-table>`
```

返回对表的每次写入的出处信息，包括操作、用户等。  表历史记录会保留 30 天。

有关详细信息，请查看[描述历史记录](../../../../delta/delta-utility.md#delta-history)。