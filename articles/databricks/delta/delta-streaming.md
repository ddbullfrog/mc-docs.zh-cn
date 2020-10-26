---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/15/2020
title: 表流式读取和写入 - Azure Databricks
description: 了解如何将 Delta 表用作流式处理源和接收器。
ms.openlocfilehash: 33201abb0d6a31747402b7cee5bb28476a3c4186
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121900"
---
# <a name="table-streaming-reads-and-writes"></a>表流读取和写入

Delta Lake 通过 `readStream` 和 `writeStream` 与 [Spark 结构化流式处理](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)深度集成。 Delta Lake 克服了通常与流式处理系统和文件相关的许多限制，包括：

* 合并低延迟引入生成的小文件

* 保持对多个流（或并发批处理作业）执行“仅一次”处理
* 使用文件作为流源时，可以有效地发现哪些文件是新文件

## <a name="delta-table-as-a-stream-source"></a><a id="delta-table-as-a-stream-source"> </a><a id="stream-source"> </a>用作流源的 Delta 表

将 Delta 表作为流源加载并在流式处理查询中使用它时，该查询将处理表中存在的所有数据以及流启动后到达的所有新数据。

可以将路径和表都作为流加载。

```scala
spark.readStream.format("delta").load("/mnt/delta/events")
```

或

```scala
spark.readStream.format("delta").table("events")
```

也可执行以下操作：

* 通过设置 `maxFilesPerTrigger` 选项，控制 Delta Lake 提供以进行流式处理的任何微批处理的最大大小。 这会指定每个触发器中要考虑的新文件的最大数量。 默认值为 1000。
* 通过设置 `maxBytesPerTrigger` 选项来限制每个微批处理中处理多少数据的速率。 这会设置“软最大值”，即批处理大约可以处理这一数量的数据，并且可能处理超出该限制的数据量。 如果将 `Trigger.Once` 用于流式处理，此选项将被忽略。 如果将此选项与 `maxFilesPerTrigger` 结合使用，则微批处理将处理数据，直到达到 `maxFilesPerTrigger` 或 `maxBytesPerTrigger` 限制。

### <a name="ignore-updates-and-deletes"></a>忽略更新和删除

结构化流式处理不处理非追加的输入，并且会在对用作源的表进行了任何修改时引发异常。 可以通过两种主要策略处理无法自动向下游传播的更改：

* 可以删除输出和检查点，并从头开始重启流。
* 可以设置以下两个选项之一：
  * `ignoreDeletes`：忽略在分区边界删除数据的事务。
  * `ignoreChanges`：如果由于数据更改操作（例如 `UPDATE`、`MERGE INTO`、分区内的 `DELETE` 或 `OVERWRITE`）而不得不在源表中重写文件，则重新处理更新。 未更改的行仍可能发出，因此下游使用者应该能够处理重复项。 删除不会传播到下游。 `ignoreChanges` 包括 `ignoreDeletes`。 因此，如果使用 `ignoreChanges`，则流不会因源表的删除或更新而中断。

#### <a name="example"></a>示例

例如，假设你有一个表 `user_events`，其中包含 `date`、`user_email` 和 `action` 列，并按 `date` 对该表进行了分区。 从 `user_events` 表向外进行流式处理，由于 GDPR 的原因，需要从中删除数据。

在分区边界（即 `WHERE` 位于分区列上）执行删除操作时，文件已经按值进行了分段，因此删除操作直接从元数据中删除这些文件。 因此，如果只想删除某些分区中的数据，则可以使用：

```scala
events.readStream
  .format("delta")
  .option("ignoreDeletes", "true")
  .load("/mnt/delta/user_events")
```

但是，如果必须基于 `user_email` 删除数据，则需要使用：

```scala
events.readStream
  .format("delta")
  .option("ignoreChanges", "true")
  .load("/mnt/delta/user_events")
```

如果使用 `UPDATE` 语句更新 `user_email`，则包含相关 `user_email` 的文件将被重写。 使用 `ignoreChanges` 时，新记录将与同一文件中的所有其他未更改记录一起传播到下游。 逻辑应该能够处理这些传入的重复记录。

### <a name="specify-initial-position"></a>指定初始位置

> [!NOTE]
>
> 此功能在 Databricks Runtime 7.3 及更高版本上可用。

可以使用以下选项来指定 Delta Lake 流式处理源的起点，而无需处理整个表。

* `startingVersion`：要从其开始的 Delta Lake 版本。 从此版本（含）开始的所有表更改都将由流式处理源读取。 可以从命令 [`DESCRIBE HISTORY events`](delta-utility.md#delta-history) 输出的 `version` 列中获取提交版本。
* `startingTimestamp`：要从其开始的时间戳。 在该时间戳（含）或之后提交的所有表更改都将由流式处理源读取。 它可以是以下任一项：
  * `'2018-10-18T22:15:12.013Z'`，即可以强制转换为时间戳的字符串
  * `cast('2018-10-18 13:36:32 CEST' as timestamp)`
  * `'2018-10-18'`，即日期字符串
  * 本身就是时间戳或可强制转换为时间戳的任何其他表达式，例如 `current_timestamp() - interval 12 hours`、`date_sub(current_date(), 1)`。

不能同时设置这两个选项，只需使用其中一个选项即可。 这两个选项仅在启动新的流式处理查询时才生效。 如果流式处理查询已启动且已在其检查点中记录进度，这些选项将被忽略。

> [!IMPORTANT]
>
> 虽然可以从指定的版本或时间戳启动流式处理源，但流式处理源的架构始终是 Delta 表的最新架构。 必须确保在指定版本或时间戳之后，不对 Delta 表进行任何不兼容的架构更改。 否则，使用错误的架构读取数据时，流式处理源可能会返回不正确的结果。

#### <a name="example"></a>示例

例如，假设你有一个表 `user_events`。 如果要从版本 5 开始读取更改，可以使用：

```scala
events.readStream
  .format("delta")
  .option("startingVersion", "5")
  .load("/mnt/delta/user_events")
```

如果想了解自 2018 年 10 月 18 日以来进行的更改，可以使用：

```scala
events.readStream
  .format("delta")
  .option("startingTimestamp", "2018-10-18")
  .load("/mnt/delta/user_events")
```

## <a name="delta-table-as-a-sink"></a><a id="delta-table-as-a-sink"> </a><a id="stream-sink"> </a>用作接收器的 Delta 表

你也可以使用结构化流式处理将数据写入 Delta 表。 即使有针对表并行运行的其他流或批处理查询，Delta Lake 也可通过事务日志确保“仅一次”处理。

### <a name="append-mode"></a><a id="append-mode"> </a><a id="streaming-append"> </a>追加模式

默认情况下，流在追加模式下运行，这会将新记录添加到表中。

可以使用路径方法：

#### <a name="python"></a>Python

```python
events.writeStream
  .format("delta")
  .outputMode("append")
  .option("checkpointLocation", "/delta/events/_checkpoints/etl-from-json")
  .start("/delta/events")
```

#### <a name="scala"></a>Scala

```scala
events.writeStream
  .format("delta")
  .outputMode("append")
  .option("checkpointLocation", "/mnt/delta/events/_checkpoints/etl-from-json")
  .start("/mnt/delta/events")
```

或表方法：

#### <a name="python"></a>Python

```python
events.writeStream
  .format("delta")
  .outputMode("append")
  .option("checkpointLocation", "/delta/events/_checkpoints/etl-from-json")
  .table("events")
```

#### <a name="scala"></a>Scala

```scala
events.writeStream
  .outputMode("append")
  .option("checkpointLocation", "/mnt/delta/events/_checkpoints/etl-from-json")
  .table("events")
```

### <a name="complete-mode"></a>完整模式

你还可以使用结构化流式处理将整个表替换为每个批。  一个示例用例是使用聚合来计算摘要：

```scala
spark.readStream
  .format("delta")
  .load("/mnt/delta/events")
  .groupBy("customerId")
  .count()
  .writeStream
  .format("delta")
  .outputMode("complete")
  .option("checkpointLocation", "/mnt/delta/eventsByCustomer/_checkpoints/streaming-agg")
  .start("/mnt/delta/eventsByCustomer")
```

上述示例持续更新包含按客户划分的事件总数的表。

对于延迟要求较为宽松的应用程序，可以使用一次性触发器来节省计算资源。 使用这些触发器按给定计划更新汇总聚合表，从而仅处理自上次更新以来收到的新数据。