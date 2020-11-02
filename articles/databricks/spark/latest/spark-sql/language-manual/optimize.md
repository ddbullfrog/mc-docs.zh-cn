---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/19/2020
title: 优化（Azure Databricks 上的 Delta Lake）- Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Delta Lake SQL 语言的 OPTIMIZE 语法优化 Delta Lake 数据的布局。
ms.openlocfilehash: 31a3f403fac4dac2956dfea6607d9b8cc442e97e
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473012"
---
# <a name="optimize--delta-lake-on-azure-databricks"></a><a id="optimize--delta-lake-on-azure-databricks"> </a><a id="optimize-delta-table"> </a>优化（Azure Databricks 上的 Delta Lake）

```sql
OPTIMIZE [db_name.]table_name [WHERE predicate]
  [ZORDER BY (col_name1, col_name2, ...)]
```

优化 Delta Lake 数据的布局。 （可选）优化数据子集或按列归置数据。 如果未指定归置，则将执行二进制打包优化。

> [!NOTE]
>
> * 二进制打包优化幂等，这意味着如果在同一数据集上运行两次，则第二次运行不起作用。 它旨在根据文件在磁盘上的大小生成均衡的数据文件，但不一定是每个文件的元组数。 但是，这两个度量值通常是相关的。
> * Z 排序不是幂等的，而应该是增量操作。 多次运行不能保证 Z 排序所需的时间减少。 但是，如果没有将新数据添加到刚刚进行 Z 排序的分区，则该分区的另一个 Z 排序将不会产生任何效果。 它旨在根据元组的数量生成均衡的数据文件，但不一定是磁盘上的数据大小。 这两个度量值通常是相关的，但可能会有例外的情况，导致优化任务时间出现偏差。  * 若要控制输出文件大小，请设置 [Spark配置](../../../../clusters/configure.md#spark-config) `spark.databricks.delta.optimize.maxFileSize`。 默认值为 `1073741824`。 指定值 `134217728` 会将最大输出文件大小设置为 100 MB。

**`WHERE`**

优化与给定分区谓词匹配的行子集。 仅支持涉及分区键属性的筛选器。

**`ZORDER BY`**

将列信息并置在同一组文件中。 Delta Lake 数据跳过算法会使用并置，大幅减少需要读取的数据量。 可以将 `ZORDER BY` 的多个列指定为以逗号分隔的列表。 但是，区域的有效性会随每个附加列一起删除。

## <a name="examples"></a>示例

```sql
OPTIMIZE events

OPTIMIZE events WHERE date >= '2017-01-01'

OPTIMIZE events
WHERE date >= current_timestamp() - INTERVAL 1 day
ZORDER BY (eventType)
```