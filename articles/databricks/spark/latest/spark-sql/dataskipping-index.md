---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: “数据跳过”索引 - Azure Databricks
description: 了解如何使用 Azure Databricks SQL 语言的 DATASKIPPING INDEX 语法。
ms.openlocfilehash: 7fd27b48f0a7c789692d3c6f69a2539102d655ea
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472853"
---
# <a name="data-skipping-index"></a>“数据跳过”索引

> [!IMPORTANT]
>
> Databricks Runtime 7.0 中已经删除了 `DATASKIPPING INDEX`。 建议改用 Delta 表，它提供了[经过改进的数据跳过功能](../../../delta/optimizations/file-mgmt.md#delta-data-skipping)。

## <a name="description"></a>说明

除了[删除分区](../../../data/tables.md#partition-pruning)之外，Databricks Runtime 还有另一项功能可用于避免扫描不相关数据，即“数据跳过”索引。 该功能使用文件级统计信息，以便在文件粒度执行更多的跳过。 此功能适用于（但不依赖于）Hive 样式分区。

“数据跳过”的有效性如何，取决于数据的特征以及数据的物理布局。 由于跳过是在文件粒度执行的，因此，将数据在多个文件之间水平进行分区是很重要的。 发生这种情况通常是因为有多个追加作业、执行（随机）分区操作、执行 Bucket 存储操作，以及/或者使用了 `spark.sql.files.maxRecordsPerFile`。 此功能不仅最适用于具有已排序的 Bucket 的表 (`df.write.bucketBy(...).sortBy(...).saveAsTable(...)` / `CREATE TABLE ... CLUSTERED BY ... SORTED BY ...`) 或具有与分区键（例如 `brandName - modelName`、`companyID - stockPrice`）相关的列的表，而且也最适用于数据正好表现出某些排序性/群聚性（例如 `orderID`、`bitcoinValue`）的情况。

> [!NOTE]
>
> 此 beta 版本功能具有许多重要限制：
>
> * 此功能是可选的：它需要基于单个表手动启用。
> * 此功能仅适用于 SQL：没有适用于此功能的数据帧 API。
> * 为表编制索引后，在对索引显式执行 REFRESH 操作之前，无法保证后续 `INSERT` 或 `ADD PARTITION` 操作的效果是可见的。

## <a name="sql-syntax"></a>SQL 语法

### <a name="create-index"></a>创建索引

```sql
CREATE DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
```

针对所提供的表为前（即最左侧）N 个受支持的列启用“数据跳过”，其中 N 由 `spark.databricks.io.skipping.defaultNumIndexedCols` 控制（默认值 ：32）

`partitionBy` 列始终会被编制索引，不计入此 N 值。

### <a name="create-index-for-columns"></a>为列创建索引

```sql
CREATE DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
    FOR COLUMNS (col1, ...)
```

针对所提供的表为指定的列的列表启用“数据跳过”。 与上述情形相同，除了指定的列之外，所有的 `partitionBy` 列也始终都会被编制索引。

### <a name="describe-index"></a>描述索引

```sql
DESCRIBE DATASKIPPING INDEX [EXTENDED] ON [TABLE] [db_name.]table_name
```

显示所提供的表中哪些列已被编制索引，并且显示所收集的相应类型的文件级统计信息。

如果指定了 `EXTENDED`，会显示名为“effectiveness_score”的第三列。该列提供了一个近似的度量，用于表示我们预期 DataSkipping 有益于相应列上筛选器的程度。

### <a name="refresh-full-index"></a>刷新完整索引

```sql
REFRESH DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
```

重新生成整个索引。 即，将为表的所有分区重新编制索引。

### <a name="refresh-partitions"></a>刷新分区

```sql
REFRESH DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
    PARTITION (part_col_name1[=val1], part_col_name2[=val2], ...)
```

只为指定的分区重新编制索引。 此操作的速度通常应该会比完整索引刷新的速度快。

### <a name="drop-index"></a>删除索引

```sql
DROP DATASKIPPING INDEX ON [TABLE] [db_name.]table_name
```

对所提供的表禁用“数据跳过”，并删除所有索引数据。