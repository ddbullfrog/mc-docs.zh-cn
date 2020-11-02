---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: 描述表 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark 和 Delta Lake SQL 语言的 DESCRIBE TABLE 语法。
ms.openlocfilehash: ecb21f1102d5bbadf1a1e3b20f765b996bb91aff
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473001"
---
# <a name="describe-table"></a>描述表

## <a name="describe-table"></a>描述表

```sql
DESCRIBE [EXTENDED] [db_name.]table_name
```

**Azure Databricks 上的 Delta Lake**

```sql
DESCRIBE [EXTENDED] delta.`<path-to-table>`
```

返回现有表的元数据（列名称、数据类型和注释）。 如果该表不存在，则会引发异常。

**`EXTENDED`**

显示有关表的详细信息，包括父数据库、表类型、存储信息和属性。

## <a name="describe-partition"></a>描述分区

```sql
DESCRIBE [EXTENDED] [db_name.]table_name PARTITION partition_spec
```

**Azure Databricks 上的 Delta Lake**

```sql
DESCRIBE [EXTENDED] delta.`<path-to-table>` PARTITION partition_spec
```

返回指定分区的元数据。 `partition_spec` 必须提供所有分区列的值。

**`EXTENDED`**

显示有关表和分区特定存储信息的基本信息。

## <a name="describe-columns"></a>描述列

```sql
DESCRIBE [EXTENDED] [db_name.]table_name column_name
```

**Azure Databricks 上的 Delta Lake**

```sql
DESCRIBE [EXTENDED] delta.`<path-to-table>`
```

返回指定列的元数据。

**`EXTENDED`**

显示有关指定列的详细信息，包括命令 `ANALYZE TABLE table_name COMPUTE STATISTICS FOR COLUMNS column_name [column_name, ...]` 收集的列统计信息。

## <a name="describe-formatted"></a>描述格式

```sql
DESCRIBE FORMATTED [db_name.]table_name
```

**Azure Databricks 上的 Delta Lake**

```sql
DESCRIBE FORMATTED delta.`<path-to-table>`
```

返回表格式。

## <a name="describe-detail-delta-lake-on-azure-databricks"></a>描述详细信息（Azure Databricks 上的 Delta Lake）

```sql
DESCRIBE DETAIL [db_name.]table_name

DESCRIBE DETAIL delta.`<path-to-table>`
```

返回架构、分区、表大小等方面的信息。 例如，你可以查看表的当前[读取器和编写器的版本](../../../../delta/versioning.md#table-version)。