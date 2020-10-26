---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/29/2020
title: 最佳做法 - Azure Databricks
description: 了解使用 Delta Lake 的最佳做法。
ms.openlocfilehash: d88a24d49c1c457e71e66e5781e4ab0eb70df213
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121837"
---
# <a name="best-practices"></a>最佳实践

本文介绍了使用 Delta Lake 时的最佳做法。

## <a name="provide-data-location-hints"></a>提供数据位置提示

如果希望在查询谓词中常规使用某一列，并且该列具有较高的基数（即包含多个非重复值），则使用 `Z-ORDER BY`。 Delta Lake 根据列值自动在文件中布局数据，在查询时根据布局信息跳过不相关的数据。

有关详细信息，请参阅 [Z 排序（多维聚类）](optimizations/file-mgmt.md#delta-zorder)。

## <a name="choose-the-right-partition-column"></a>选择正确的分区列

可以按列对 Delta 表进行分区。 最常使用的分区列是 `date`。
请按照以下两个经验法则来确定要根据哪个列进行分区：

* 如果某个列的基数将会很高，则不要将该列用于分区。 例如，如果你按列 `userId` 进行分区并且可能有 100 万个不同的用户 ID，则这是一种错误的分区策略。
* 每个分区中的数据量：如果你预计该分区中的数据至少有 1 GB，可以按列进行分区。

## <a name="compact-files"></a><a id="compact-files"> </a><a id="delta-compact-files"> </a>压缩文件

如果你连续将数据写入到 Delta 表，则它会随着时间的推移累积大量文件，尤其是小批量添加数据时。 这可能会对表读取效率产生不利影响，并且还会影响文件系统的性能。 理想情况下，应当将大量小文件定期重写到较小数量的较大型文件中。 这称为压缩。

你可以使用 [OPTIMIZE](optimizations/file-mgmt.md#delta-optimize) 命令来压缩表。

## <a name="replace-the-content-or-schema-of-a-table"></a><a id="delta-replace-table"> </a><a id="replace-the-content-or-schema-of-a-table"> </a>替换表的内容或架构

有时候，你可能希望替换 Delta 表。 例如： 。

* 你发现表中的数据不正确，需要对内容进行替换。
* 你希望重写整个表，以执行不兼容架构更改（删除列或更改列类型）。

尽管可以删除 Delta 表的整个目录并在同一路径上创建新表，但不建议这样做，因为：

* 删除目录效率不高。 删除某个包含极大文件的目录可能需要数小时甚至数天的时间。
* 删除的文件中的所有内容都会丢失；如果删除了错误的表，则很难恢复。
* 目录删除不是原子操作。 删除表时，某个读取表的并发查询可能会失败或看到的是部分表。

如果不需要更改表架构，则可以从 Delta 表中[删除](delta-update.md#delete-from-a-table)数据并插入新数据，或者通过[更新](delta-update.md#update-a-table)表来纠正不正确的值。

如果要更改表架构，则能够以原子方式替换整个表。 例如： 。

### <a name="python"></a>Python

```python
dataframe.write \
  .format("delta") \
  .mode("overwrite") \
  .option("overwriteSchema", "true") \
  .partitionBy(<your-partition-columns>) \
  .saveAsTable("<your-table>") # Managed table
dataframe.write \
  .format("delta") \
  .mode("overwrite") \
  .option("overwriteSchema", "true") \
  .option("path", "<your-table-path>") \
  .partitionBy(<your-partition-columns>) \
  .saveAsTable("<your-table>") # External table
```

### <a name="sql"></a>SQL

```sql
REPLACE TABLE <your-table> USING DELTA PARTITIONED BY (<your-partition-columns>) AS SELECT ... -- Managed table
REPLACE TABLE <your-table> USING DELTA PARTITIONED BY (<your-partition-columns>) LOCATION "<your-table-path>" AS SELECT ... -- External table
```

### <a name="scala"></a>Scala

```scala
dataframe.write
  .format("delta")
  .mode("overwrite")
  .option("overwriteSchema", "true")
  .partitionBy(<your-partition-columns>)
  .saveAsTable("<your-table>") // Managed table
dataframe.write
  .format("delta")
  .mode("overwrite")
  .option("overwriteSchema", "true")
  .option("path", "<your-table-path>")
  .partitionBy(<your-partition-columns>)
  .saveAsTable("<your-table>") // External table
```

此方法有多个优点：

* 覆盖表的速度要快得多，因为它不需要以递归方式列出目录或删除任何文件。
* 表的旧版本仍然存在。 如果删除了错误的表，则可以使用[按时间顺序查看](delta-batch.md#query-an-older-snapshot-of-a-table-time-travel)轻松检索旧数据。
* 这是一个原子操作。 在删除表时，并发查询仍然可以读取表。
* 由于 Delta Lake ACID 事务保证，如果覆盖表失败，则该表将处于其以前的状态。

此外，如果你想要在覆盖表后删除旧文件以节省存储成本，则可以使用 [VACUUM](delta-utility.md#delta-vacuum) 来删除它们。 它针对文件删除进行了优化，通常比删除整个目录要快。