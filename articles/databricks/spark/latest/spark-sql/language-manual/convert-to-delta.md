---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: Convert To Delta（Azure Databricks 上的 Delta Lake）- Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 CONVERT TO DELTA 语法。
ms.openlocfilehash: b09e0795f0c0c2d2c4b3de58975caca0d9631846
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473039"
---
# <a name="convert-to-delta-delta-lake-on-azure-databricks"></a>转换为 Delta（Azure Databricks 上的 Delta Lake）

```sql
CONVERT TO DELTA [ [db_name.]table_name | parquet.`<path-to-table>` ] [NO STATISTICS]
[PARTITIONED BY (col_name1 col_type1, col_name2 col_type2, ...)]
```

> [!NOTE]
>
> `CONVERT TO DELTA [db_name.]table_name` 需要 Databricks Runtime 6.6 或更高版本。

将现有 Parquet 表就地转换为 Delta 表。 此命令会列出目录中的所有文件，创建 Delta Lake 事务日志来跟踪这些文件，并通过读取所有 Parquet 文件的页脚来自动推断数据架构。 转换过程会收集统计信息，以提升转换后的 Delta 表的查询性能。 如果提供表名，则元存储也将更新，以反映该表现在是 Delta 表。

**`NO STATISTICS`**

在转换过程中绕过统计信息收集，以更快的速度完成转换。 将表转换为 Delta Lake 后，可以使用 `OPTIMIZE ZORDER BY` 重新组织数据布局并生成统计信息。

**`PARTITIONED BY`**

按指定的列对创建的表进行分区。 如果数据已分区，则为必需。  如果目录结构不符合 `PARTITIONED BY` 规范，转换过程将中止并引发异常。 如果未提供 `PARTITIONED BY` 子句，该命令便假定该表未分区。

## <a name="caveats"></a>注意事项

Delta Lake 未跟踪的文件均不可见，运行 `VACUUM` 时可将其删除。 在转换过程中，请勿更新或追加数据文件。 转换表后，请确保通过 Delta Lake 执行所有写入。

多个外部表可能都使用同一个基础 Parquet 目录。 在这种情况下，如果在其中某个外部表上运行 `CONVERT`，则无法访问其他外部表，因为其基础目录已从 Parquet 转换为 Delta Lake。 若要再次查询或写入这些外部表，还必须在这些表上运行 `CONVERT`。

`CONVERT` 会将目录信息（例如架构和表属性）填充到 Delta Lake 事务日志中。 如果基础目录已转换为 Delta Lake 并且其元数据与目录元数据不同，则会引发 `convertMetastoreMetadataMismatchException`。 如果希望 `CONVERT` 覆盖 Delta Lake 事务日志中的现有元数据，请将 SQL 配置 `spark.databricks.delta.convert.metadataCheck.enabled` 设置为 false。

## <a name="undo-the-conversion"></a>撤消转换

如果执行了可更改数据文件的 Delta Lake 操作（例如 `DELETE` 或 `OPTIMIZE`），请先运行以下命令进行垃圾回收：

```sql
VACUUM delta.`<path-to-table>` RETAIN 0 HOURS
```

然后，删除 `<path-to-table>/_delta_log` 目录。