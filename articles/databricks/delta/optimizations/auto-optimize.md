---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 自动优化 - Azure Databricks
description: 了解 Azure Databricks 上的 Delta Lake 如何自动压缩文件并优化到 Delta 表的写入。
ms.openlocfilehash: 42b58e8a116ce0cb68b9bb6fdcc8a48530006a60
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121853"
---
# <a name="auto-optimize"></a>自动优化

自动优化是一组可选功能，可在向 Delta 表进行各次写入时自动压缩小文件。 在写入过程中支付少量的开销可以为进行频繁查询的表带来显著的好处。  自动优化在下列情况下尤其有用：

* 可以接受分钟级延迟的流式处理用例
* `MERGE INTO` 是写入到 Delta Lake 的首选方法
* `CREATE TABLE AS SELECT` 或 `INSERT INTO` 是常用操作

## <a name="how-auto-optimize-works"></a>自动优化的工作原理

自动优化包括两个互补功能：优化写入和自动压缩。

**优化写入**

Azure Databricks 基于实际数据动态优化 Apache Spark 分区大小，并尝试为每个表分区写出 128 MB 的文件。 这是一个近似大小，可能因数据集特征而异。

**自动压缩**

在每次写入后，Azure Databricks 会检查文件是否可以进一步压缩，并运行一个快速 `OPTIMIZE` 作业（128 MB 文件大小，而不是 1 GB），以便进一步压缩包含最多小文件的分区的文件。

> [!div class="mx-imgBorder"]
> ![优化写入](../../_static/images/delta/optimized-writes.png)

## <a name="usage"></a>使用情况

自动优化设计为针对特定的 Delta 表进行配置。 你可以通过设置[表属性](../delta-batch.md#table-properties) `delta.autoOptimize.optimizeWrite = true` 为表启用优化写入。 同样，你可以设置 `delta.autoOptimize.autoCompact = true` 来启用自动压缩。

* 对于现有表，请运行：

  ```sql
  ALTER TABLE [table_name | delta.`<table-path>`] SET TBLPROPERTIES (delta.autoOptimize.optimizeWrite = true, delta.autoOptimize.autoCompact = true)
  ```

* 若要确保所有新的 Delta 表都启用了这些功能，请设置以下 SQL 配置：

  ```sql
  set spark.databricks.delta.properties.defaults.autoOptimize.optimizeWrite = true;
  set spark.databricks.delta.properties.defaults.autoOptimize.autoCompact = true;
  ```

此外，你可以使用以下配置为 Spark 会话启用和禁用这两项功能：

* `spark.databricks.delta.optimizeWrite.enabled`
* `spark.databricks.delta.autoCompact.enabled`

会话配置优先于表属性，因此你可以更好地控制何时选择加入或选择退出这些功能。

## <a name="when-to-opt-in-and-opt-out"></a>何时选择加入和选择退出

本部分提供了有关何时选择加入和选择退出自动优化功能的指导。

### <a name="optimized-writes"></a>优化写入

优化写入旨在最大程度地提高写入到存储服务的数据的吞吐量。 这可以通过减少写入的文件数来实现，而不会牺牲过多的并行度。

优化写入需要根据目标表的分区结构来混排数据。 这种混排自然会产生额外成本。 但是，写入过程中的吞吐量增加可能会对冲掉混排成本。 如果没有，查询数据时吞吐量会增加，因此这个功能仍然是很有价值的。

优化写入的关键在于它是一个自适应的混排。 如果你有一个流式处理引入用例，并且输入数据速率随时间推移而变化，则自适应混排会根据不同微批次的传入数据速率自行进行相应调整。 如果代码片段在写出流之前执行 `coalesce(n)` 或 `repartition(n)`，则可以删除这些行。

#### <a name="when-to-opt-in"></a>何时选择加入

* 可以接受分钟级延迟的流式处理用例
* 使用 `MERGE`、`UPDATE`、`DELETE`、`INSERT INTO`、`CREATE TABLE AS SELECT` 之类的 SQL 命令时

#### <a name="when-to-opt-out"></a>何时选择退出

当写入的数据以兆字节为数量级且存储优化实例不可用时。

### <a name="auto-compaction"></a>自动压缩

自动压缩发生在向表进行写入的操作成功后，在执行了写入操作的群集上同步运行。 这意味着，如果你的代码模式向 Delta Lake 进行写入，然后立即调用 `OPTIMIZE`，则可以在启用自动压缩的情况下删除 `OPTIMIZE` 调用。

自动压缩使用与 `OPTIMIZE` 不同的试探法。 由于它在写入后同步运行，因此我们已将自动压缩功能优化为使用以下属性运行：

* Azure Databricks 不支持将 Z 排序与自动压缩一起使用，因为 Z 排序的成本要远远高于纯压缩。
* 自动压缩生成比 `OPTIMIZE` (1 GB) 更小的文件 (128 MB)。
* 自动压缩“贪婪地”选择能够最好地利用压缩的有限的一组分区。 所选的分区数因其启动时所处的群集的大小而异。 如果群集有较多的 CPU，则可以优化较多的分区。

#### <a name="when-to-opt-in"></a>何时选择加入

* 可以接受分钟级延迟的流式处理用例
* 当表上没有常规 `OPTIMIZE` 调用时

#### <a name="when-to-opt-out"></a>何时选择退出

当其他编写器可能同时执行 `DELETE`、`MERGE`、`UPDATE` 或 `OPTIMIZE` 之类的操作时（因为自动压缩可能会导致这些作业发生事务冲突）。 如果自动压缩由于事务冲突而失败，Azure Databricks 不会使压缩失败，也不会重试压缩。

## <a name="example-workflow-streaming-ingest-with-concurrent-deletes-or-updates"></a>示例工作流：使用并发删除或更新操作进行流式引入

此工作流假定你有一个群集运行全天候流式处理作业来引入数据，另一个群集每小时、每天或临时运行一次作业来删除或更新一批记录。 对于此用例，Azure Databricks 建议你：

* 使用以下命令在表级别启用优化写入

  ```sql
  ALTER TABLE <table_name|delta.`table_path`> SET TBLPROPERTIES (delta.autoOptimize.optimizeWrite = true)
  ```

  这可以确保流写入的文件数以及删除和更新作业的大小是最佳的。

* 对执行删除或更新操作的作业使用以下设置将在会话级别启用自动压缩。

  ```scala
  spark.sql("set spark.databricks.delta.autoCompact.enabled = true")
  ```

   这样就可以在表中压缩文件。 由于这发生在删除或更新之后，因此可以减轻事务冲突的风险。

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**自动优化是否对文件进行 Z 排序？**

自动优化仅对小文件执行压缩。 它不对文件进行 [Z 排序](file-mgmt.md#delta-zorder)。

**自动优化是否会损坏 Z 排序的文件？**

自动优化会忽略 Z 排序的文件。 它仅压缩新文件。

**如果我对要将数据流式传输到其中的表启用了自动优化，并且某个并发事务与优化冲突，那么我的作业是否会失败？**

否。 系统会忽略导致自动优化失败的事务冲突，流将继续正常运行。

**如果在我的表上启用了自动优化，是否需要计划 `OPTIMIZE` 作业？**

对于大于 10 TB 的表，建议让 `OPTIMIZE` 按计划运行，以便进一步合并文件并减少 Delta 表的元数据。 由于自动优化不支持 Z 排序，因此仍应将 `OPTIMIZE ... ZORDER BY` 作业计划为定期运行。