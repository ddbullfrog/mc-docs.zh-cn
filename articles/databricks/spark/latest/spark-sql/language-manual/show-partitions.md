---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/27/2020
title: 显示分区 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 SHOW PARTITIONS 语法。
ms.openlocfilehash: 81ae724640c98f44c0a78210edf04e5e2596c7dc
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472747"
---
# <a name="show-partitions"></a>显示分区

```sql
SHOW PARTITIONS [db_name.]table_name [PARTITION part_spec]

part_spec:
  : (part_col_name1=val1, part_col_name2=val2, ...)
```

列出表的分区，按给定的分区值进行筛选。 启用 Hive 支持后，只有使用 Delta Lake 格式或 Hive 格式创建的表才支持列出分区。