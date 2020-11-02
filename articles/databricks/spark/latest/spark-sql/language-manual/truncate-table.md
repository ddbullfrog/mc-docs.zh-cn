---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/02/2020
title: 截断表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark 和 Delta Lake SQL 语言的 TRUNCATE TABLE 语法。
ms.openlocfilehash: ddc8e61f89b0ad07153dce2807d035261a20e9a1
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472941"
---
# <a name="truncate-table"></a>截断表

```sql
TRUNCATE TABLE table_name [PARTITION part_spec]

part_spec:
  : (part_col1=value1, part_col2=value2, ...)
```

删除表中的所有行或表中匹配的分区。 该表不得为外部表或视图。

**`PARTITION`**

用于匹配要截断的分区的部分分区规范。 在 Spark 2.0 中，只有使用 Hive 格式创建的表才支持此功能。 从 Spark 2.1 开始，还支持数据源表。 Delta 表不支持。