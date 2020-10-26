---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: 通过文件管理优化性能 - Azure Databricks
description: 了解 Azure Databricks 上的 Delta Lake 可用的文件管理机制以提高性能。
ms.openlocfilehash: f9d16cd1aa6bfccb5dfafda07235019cff85feb4
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121941"
---
# <a name="optimize-performance-with-file-management"></a>通过文件管理优化性能

为提高查询速度，Azure Databricks 上的 Delta Lake 支持优化存储在云存储中的数据的布局。 Azure Databricks 上的 Delta Lake 支持两种布局算法：二进制打包和 Z 排序。

本文介绍如何运行优化命令、两种布局算法的工作原理以及如何清理过时的表快照。

* [常见问题解答](#optimize-faq)阐释了为什么优化不是自动进行的，并提供了有关运行优化命令的频率的建议。
* 有关演示优化优点的笔记本，请参阅[优化示例](optimization-examples.md)。
* 有关 Azure Databricks SQL 上的 Delta Lake 优化命令的参考信息，请参阅[优化（Azure Databricks 上的 Delta Lake）](../../spark/latest/spark-sql/language-manual/optimize.md)。

## <a name="compaction-bin-packing"></a><a id="compaction-bin-packing"> </a> <a id="delta-optimize"> </a>压缩（二进制打包）

Azure Databricks 上的 Delta Lake 可以将小文件合并为较大的文件，从而提高表中读取查询的速度。 通过运行 `OPTIMIZE` 命令触发压缩：

```sql
OPTIMIZE delta.`/data/events`
```

或

```sql
OPTIMIZE events
```

如果拥有大量数据，并且只想要优化其中的一个子集，则可以使用 `WHERE` 指定一个可选的分区谓词：

```sql
OPTIMIZE events WHERE date >= '2017-01-01'
```

> [!NOTE]
>
> * 二进制打包优化幂等，这意味着如果在同一数据集上运行两次，则第二次运行不起作用。
> * 二进制打包旨在根据其在磁盘上的大小生成均匀平衡的数据文件，但不一定是每个文件的元组数。 但是，这两个度量值通常是相关的。

Delta 表的读取器使用快照隔离，这意味着，当 `OPTIMIZE` 从事务日志中删除不必要的文件时，它们不会中断。 `OPTIMIZE` 不会对表进行任何数据相关更改，因此，在 `OPTIMIZE` 之前和之后读取都具有相同的结果。 对作为流式处理源的表执行 `OPTIMIZE` 不会影响将此表视为源的任何当前或未来的流。 `OPTIMIZE` 返回所删除文件的文件统计信息（最小值、最大值、总计等）和操作添加的文件。 优化统计信息还包含 Z 排序统计信息、批处理数和已优化分区数。

> [!NOTE]
>
> 在 Databricks Runtime 6.0 及更高版本中可用。

你还可以使用[自动优化](auto-optimize.md)自动压缩小文件。

## <a name="data-skipping"></a><a id="data-skipping"> </a><a id="delta-data-skipping"> </a>跳过数据

向 Delta 表中写入数据时，会自动收集跳过数据信息。 Azure Databricks 上的 Delta Lake 会在查询时利用此信息（最小值和最大值）来提供更快的查询。 不需要配置跳过的数据；此功能会在适用时激活。 但其有效性取决于数据的布局。 为了获取最佳结果，请应用 [Z 排序](#delta-zorder)。

详细了解 Azure Databricks 上的 Delta Lake 跳过数据和 Z 排序的优点，请参阅[优化示例](optimization-examples.md)中的笔记本。 默认情况下，Azure Databricks 上的 Delta Lake 收集你的表架构中定义的前 32 列的统计信息。 你可以使用[表属性](../delta-batch.md#table-properties) `dataSkippingNumIndexedCols` 来更改此值。 在写入文件时，添加更多的列来收集统计信息会增加额外的开销。

收集长字符串的统计信息成本高昂。 若要避免收集长字符串的统计信息，可以将表属性 `dataSkippingNumIndexedCols` 配置为避免包含长字符串的列，或使用 `ALTER TABLE CHANGE COLUMN` 将包含长字符串的列移动到大于 `dataSkippingNumIndexedCols` 的列。 为了收集统计信息，嵌套列中的每个字段都被视为单独的列。

有关详细信息，请参阅博客文章：[通过 Databricks Delta 以在数秒内处理数 PB 的数据](https://databricks.com/blog/2018/07/31/processing-petabytes-of-data-in-seconds-with-databricks-delta.html)。

## <a name="z-ordering-multi-dimensional-clustering"></a><a id="delta-zorder"> </a><a id="z-ordering-multi-dimensional-clustering"> </a>Z 排序（多维聚类）

Z 排序是并置同一组文件中相关信息的[方法](https://en.wikipedia.org/wiki/Z-order_curve)。 Azure Databricks 上的 Delta Lake 数据跳过算法会自动使用此并置，大幅减少需要读取的数据量。 对于 Z 排序数据，请在 `ZORDER BY` 子句中指定要排序的列：

```sql
OPTIMIZE events
WHERE date >= current_timestamp() - INTERVAL 1 day
ZORDER BY (eventType)
```

如果希望在查询谓词中常规使用某一列，并且该列具有较高的基数（即包含多个非重复值），请使用 `ZORDER BY`。

可以将 `ZORDER BY` 的多个列指定为以逗号分隔的列表。 但是，区域的有效性会随每个附加列一起删除。 Z 排序对于未收集统计信息的列无效并且会浪费资源，因为需要列本地统计信息（如最小值、最大值和总计）才能跳过数据。 可以通过对架构中的列重新排序或增加从中收集统计信息的列数，对某些列配置统计信息收集。 （有关详细信息，请参阅[跳过数据](#delta-data-skipping)部分）。

> [!NOTE]
>
> * Z 排序不是幂等的，而应该是增量操作。 多次运行不能保证 Z 排序所需的时间减少。 但是，如果没有将新数据添加到刚刚进行 Z 排序的分区，则该分区的另一个 Z 排序将不会产生任何效果。
> * Z 排序旨在根据元组的数量生成均匀平衡的数据文件，但不一定是磁盘上的数据大小。 这两个度量值通常是相关的，但可能会有例外的情况，导致优化任务时间出现偏差。
>
>   例如，如果 `ZORDER BY` 日期，并且最新记录的宽度比过去多很多（例如数组或字符串值较长），则 `OPTIMIZE` 作业的任务持续时间和所生成文件的大小都会出现偏差。 但这只是 `OPTIMIZE` 命令本身的问题；它不应对后续查询产生任何负面影响。

## <a name="notebooks"></a>笔记本

有关优化优点的示例，请参阅以下笔记本：

* [优化示例](optimization-examples.md)
  * [Databricks 上的 Delta Lake 优化 Python 笔记本](optimization-examples.md#delta-lake-on-databricks-optimizations-python-notebook)
  * [Databricks 上的 Delta Lake 优化 Scala 笔记本](optimization-examples.md#delta-lake-on-databricks-optimizations-scala-notebook)
  * [Databricks 上的 Delta Lake 优化 SQL 笔记本](optimization-examples.md#delta-lake-on-databricks-optimizations-sql-notebook)

## <a name="improve-interactive-query-performance"></a><a id="improve-interactive-query-performance"> </a><a id="update-async"> </a>提高交互式查询性能

Delta Engine 提供了一些额外的机制来提高查询性能。

### <a name="manage-data-recency"></a>管理数据时效性

在每个查询开头，Delta 表自动更新到最新版本的表。 当命令状态报告 `Updating the Delta table's state` 时，可以在笔记本中观察到此过程。 但是，当对表运行历史分析时，你可能不需要最新的数据，尤其是在频繁引入流式处理数据的表中。 在这些情况下，可以在 Delta 表的过时快照上运行查询。 这会降低从查询获取结果的延迟时间。

可以通过将 Spark 会话配置 `spark.databricks.delta.stalenessLimit` 设置为时间字符串值（例如 `1h`、`15m`、`1d` 分别为 1 小时、15 分钟和 1 天）来配置表数据的过时程度。 此配置特定于会话，因此不会影响其他用户从其他笔记本、作业或 BI 工具访问此表。 此外，此设置不会阻止更新表；它只会阻止查询等待表更新。 此更新仍会在后台发生，并将在群集中公平地共享资源。 如果超过过期限制，则查询将在表状态更新上阻止。

### <a name="enhanced-checkpoints-for-low-latency-queries"></a><a id="enhanced-checkpoints"> </a><a id="enhanced-checkpoints-for-low-latency-queries"> </a>用于低延迟查询的增强检查点

Delta Lake 写入[检查点](https://github.com/delta-io/delta/blob/master/PROTOCOL.md#checkpoints)作为 Delta 表的聚合状态，每 10 次提交写入一次。 这些检查点作为计算表的最新状态的起点。 如果没有检查点，Delta Lake 就必须读取一个大型 JSON 文件（“delta”文件）的集合，表示提交到事务日志以计算表的状态。 此外，列级统计信息 Delta Lake 用于执行存储在检查点中的[数据跳过](#delta-data-skipping)操作。

> [!IMPORTANT]
>
> Delta Lake 检查点与[结构化流检查点](../../spark/latest/structured-streaming/production.md#recover-from-query-failures)不同。

在 Databricks Runtime 7.2 及更低的级别中，列级统计信息作为 JSON 列存储在 Delta Lake 检查点中。 在 Databricks Runtime 7.3 及更高版本中，列级统计信息作为结构。 结构格式使得 Delta Lake 读取速度快得多，因为：

* Delta Lake 不会执行昂贵的 JSON 分析来获取列级统计信息。
* Parquet 列修剪功能可以显著减少读取列的统计信息所需的 I/O。

结构格式启用一系列优化，这些优化可以将增量 Delta Lake 读取操作的开销从数秒降低到数十毫秒，大大降低短查询的延迟。

#### <a name="manage-column-level-statistics-in-checkpoints"></a>在检查点中管理列级统计信息

使用表属性 `delta.checkpoint.writeStatsAsJson` 和 `delta.checkpoint.writeStatsAsStruct` 来管理如何在检查点中写入统计信息。
如果两个表属性都为 `false`，则 Delta Lake 无法执行跳过数据。

在 Databricks Runtime 7.3 及更高版本中：

* 批处理以 JSON 格式和结构格式编写写入统计信息。 `delta.checkpoint.writeStatsAsJson` 上声明的默认值为 `true`。
* 流式处理以 JSON 格式写入写入统计信息（以最大程度地减少检查点对写入延迟的影响）。 若要同时编写结构格式，请参阅[为结构化流查询启用增强的检查点](#enhanced-ss)。
* 在这两种情况下，都默认未定义 `delta.checkpoint.writeStatsAsStruct`。
* 读取器在可用时使用结构列，否则回退到使用 JSON 列。

在 Databricks Runtime 7.2 及更旧版本中，读取器只使用 JSON 列。 因此，如果 `delta.checkpoint.writeStatsAsJson` 为 `false`，则此类读取器不能执行跳过数据。

> [!IMPORTANT]
>
> 增强的检查点不会破坏与开源 Delta Lake 读取器的兼容性。 但是，将 `delta.checkpoint.writeStatsAsJson` 设置为 `false` 可能会影响专有的 Delta Lake 读取器。 请与供应商联系，以了解有关性能影响的详细信息。

#### <a name="trade-offs-with-statistics-in-checkpoints"></a>检查点中统计信息的权衡

由于在检查点中写入统计信息会产生成本（通常小于一分钟，即使是对大表），因此需要权衡编写检查点所花的时间和与 Databricks Runtime 7.2 及更旧版本的兼容性。 如果能够将所有工作负载升级到 Databricks Runtime 7.3 或更高版本，则可以通过禁用旧版 JSON 统计信息来降低写入检查点的成本。 下表汇总了此权衡。

如果跳过数据不适用于你的应用程序，可以将这两个属性都设置为 false，这样就不会收集或写入任何统计信息。
我们不建议此配置。

> [!div class="mx-imgBorder"]
> ![统计信息权衡](../../_static/images/delta/stats-tradeoffs.png)

#### <a name="enable-enhanced-checkpoints-for-structured-streaming-queries"></a><a id="enable-enhanced-checkpoints-for-structured-streaming-queries"> </a><a id="enhanced-ss"> </a>

如果结构化流式处理工作负载没有低延迟要求（即要求延迟在一分钟以内），你可以运行以下 SQL 命令来启用增强的检查点：

```sql
ALTER TABLE [<table-name>|delta.`<path-to-table>`] SET TBLPROPERTIES
('delta.checkpoint.writeStatsAsStruct' = 'true')
```

如果不使用 Databricks Runtime 7.2 或更旧版本来查询数据，还可以通过设置以下表属性来降低检查点写入延迟：

```sql
ALTER TABLE [<table-name>|delta.`<path-to-table>`] SET TBLPROPERTIES
(
 'delta.checkpoint.writeStatsAsStruct' = 'true',
 'delta.checkpoint.writeStatsAsJson' = 'false'
)
```

#### <a name="disable-writes-from-clusters-that-write-checkpoints-without-the-stats-struct"></a>禁用写入无统计结构的检查点的群集写入

Databricks Runtime 7.2 及更旧版本的编写器会写入无统计结构的检查点，这会阻止优化 Databricks Runtime 7.3 读取器。
若要阻止运行 Databricks Runtime 7.2 及更旧版本的群集写入 Delta 表，你可以使用 `upgradeTableProtocol` 方法升级 Delta 表：

##### <a name="python"></a>Python

```python
from delta.tables import DeltaTable
delta = DeltaTable.forPath(spark, "path_to_table") # or DeltaTable.forName
delta.upgradeTableProtocol(1, 3)
```

##### <a name="scala"></a>Scala

```scala
import io.delta.tables.DeltaTable
val delta = DeltaTable.forPath(spark, "path_to_table") // or DeltaTable.forName
delta.upgradeTableProtocol(1, 3)
```

> [!WARNING]
>
> 应用 `upgradeTableProtocol` 方法可阻止运行 Databricks Runtime 7.2 及更旧版本的群集写入表，此更改不可逆。
> 建议仅在提交到新格式后才升级表。 可以通过使用 Databricks Runtime 7.3 创建表的浅层[克隆](../../spark/latest/spark-sql/language-manual/clone.md)来尝试这些优化。

升级表编写器版本后，编写器必须遵循 `'delta.checkpoint.writeStatsAsStruct'` 和 `'delta.checkpoint.writeStatsAsJson'` 的设置。

下表总结了如何在各种版本的 Databricks Runtime、表协议版本和编写器类型中利用增强的检查点。

> [!div class="mx-imgBorder"]
> ![增强的检查点](../../_static/images/delta/enhanced-checkpoints.png)

## <a name="frequently-asked-questions-faq"></a><a id="frequently-asked-questions-faq"> </a><a id="optimize-faq"> </a>常见问题解答 (FAQ)

为什么 `OPTIMIZE` 不是自动的？

`OPTIMIZE` 操作启动多个 Spark 作业，以便通过压缩优化文件大小调整（并选择性执行 Z 排序）。 由于 `OPTIMIZE` 执行的内容大多是压缩小文件，因此在此操作生效之前，你必须先积累许多小文件。 因此，`OPTIMIZE` 操作不会自动运行。

而且，运行 `OPTIMIZE`（特别是 `ZORDER`）是时间和资源成本高昂的操作。 如果 Databricks 自动运行 `OPTIMIZE` 或等待分批写入数据，则将不可运行（以 Delta 表为源的）低延迟 Delta Lake 流。 许多客户都有一个从未优化的 Delta 表，因为他们仅流式传输这些表中的数据，而享受不到 `OPTIMIZE` 可提供的查询优势。

最后，Delta Lake 会自动收集有关写入表的文件（无论是否通过 `OPTIMIZE` 操作）的统计信息。 这意味着，从 Delta 表的读取将利用此信息，无论该表或分区上是否运行了 `OPTIMIZE` 操作。

我应该多久运行一次 `OPTIMIZE`？

如果选择运行 `OPTIMIZE` 的频率，则会在性能和成本之间进行权衡。 如果希望获得更好的最终用户查询性能，则应更频繁地运行 `OPTIMIZE`（根据资源使用量，可能需要较高的成本）。 如果要优化成本，应减少运行它。

建议从每天运行一次 `OPTIMIZE` 开始。 然后在此修改作业。

运行 `OPTIMIZE`（二进制打包和 Z 排序）的最佳实例类型是什么？

这两个操作都是执行大量 Parquet 解码和编码的 CPU 密集型操作。

对于这些工作负载，建议采用 F 或 Fsv2 系列。