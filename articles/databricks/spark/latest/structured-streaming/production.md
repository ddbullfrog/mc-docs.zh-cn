---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 生产中的结构化流式处理 - Azure Databricks
description: 了解如何在 Azure Databricks 中使 Apache Spark 结构化流式处理应用程序具有更好的容错能力和性能。
ms.openlocfilehash: 706750e5ca059f34a8177fab2e7119aad8bbc41f
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473057"
---
# <a name="structured-streaming-in-production"></a>生产中的结构化流式处理

可以方便地将笔记本附加到群集并以交互方式运行流式处理查询。 但是，在生产环境中运行它们时，可能需要更高的可靠性和正常运行时间保证。 本文介绍了如何使用 Azure Databricks [作业](../../../jobs.md)使流式处理应用程序更具容错能力。

## <a name="recover-from-query-failures"></a>在发生查询失败后进行恢复

生产级流式处理应用程序必须有可靠的故障处理能力。 在结构化流式处理中，如果为流式处理查询启用检查点，则可以在失败后重启查询，重启的查询将从失败查询停止的位置继续，同时确保容错并保证数据一致性。 因此，弱要使查询具有容错性，你必须启用查询检查点并将 Databricks [作业](../../../jobs.md)配置为在失败后自动重启查询。

### <a name="enable-checkpointing"></a>启用检查点

若要启用检查点，请在启动查询之前将选项 `checkpointLocation` 设置为 DBFS 或云存储路径。 例如： 。

```scala
streamingDataFrame.writeStream
  .format("parquet")
  .option("path", "dbfs://outputPath/")
  .option("checkpointLocation", "dbfs://checkpointPath")
  .start()
```

此检查点位置保留可用于唯一标识查询的所有基本信息。
因此，每个查询都必须有不同的检查点位置，并且多个查询不应有相同的位置。 有关更多详细信息，请参阅[结构化流式处理编程指南](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)。

> [!NOTE]
>
> 尽管 `checkpointLocation` 是大多数类型的输出接收器的必需选项，但当你未提供 `checkpointLocation` 时，某些接收器（例如内存接收器）可能会在 DBFS 上自动生成一个临时检查点位置。 临时检查点位置不能确保任何容错，也不能保证数据一致性，并且可能无法进行正常的清理。
> 建议始终指定 `checkpointLocation` 选项，这是最佳做法。

### <a name="configure-jobs-to-restart-streaming-queries-on-failure"></a>将作业配置为在失败时重启流式处理查询

你可以使用包含流式处理查询的笔记本或 JAR 创建 Azure Databricks [作业](../../../jobs.md)，并将其配置为：

* 始终使用新群集。
* 始终在失败时重试。

作业与结构化流式处理 API 紧密集成，并且可以监视运行中处于活动状态的所有流式处理查询。 此配置可确保如果查询的任何部分失败，作业会自动终止此运行（以及所有其他查询），并在新群集中启动新的运行。 新运行会重新执行笔记本或 JAR 代码，并再次重启所有查询。 这是确保返回到良好状态的最安全的方法。

> [!WARNING]
>
> 长时间运行的作业不支持[笔记本工作流](../../../notebooks/notebook-workflows.md)。 因此，我们不建议在流式处理作业中使用笔记本工作流。

> [!NOTE]
>
> * 任何活动的流式处理查询中的失败都会导致活动的运行失败，并终止所有其他流式处理查询。
> * 你无需在笔记本的末尾使用 `streamingQuery.awaitTermination()` 或 `spark.streams.awaitAnyTermination()`。 当流式处理查询处于活动状态时，作业会自动防止运行完成。

下面是建议的作业配置的详细信息。

* **群集** ：将此项始终设置为使用新群集并使用最新的 Spark 版本（或至少使用版本 2.1）。 在查询和 Spark 版本升级之后，在 Spark 2.1 及更高版本中启动的查询是可恢复的查询。
* **警报** ：如果希望在失败时收到电子邮件通知，请设置此项。
* **计划** ：不设置计划。
* **Timeout** ：不设置超时。 流式处理查询运行时间无限长。
* **最大并发运行数** ：设置为 **1** 。 每个查询只能有一个并发的活动实例。
* **重试** ：设置为“不受限制”。

请参阅[作业](../../../jobs.md)来了解这些配置。 下面是良好作业配置的屏幕截图。

> [!div class="mx-imgBorder"]
> ![作业配置](../../../_static/images/spark/structured-streaming/job-conf-azure.png)

### <a name="recover-after-changes-in-a-streaming-query"></a>在流式处理查询发生更改后恢复

在从同一检查点位置进行的各次重启之间，对于流式处理查询中允许哪些更改存在限制。
下面的几种类型的更改是不允许的，或者是更改效果未明确的。 对于它们：

* “允许”一词意味着你可以执行指定的更改，但其效果的语义是否明确取决于查询和更改。
* “不允许”一词意味着不应执行指定的更改，因为重启的查询可能会失败并出现不可预知的错误。
* `sdf` 表示通过 `sparkSession.readStream` 生成的流式处理数据帧/数据集。

#### <a name="types-of-changes"></a>更改的类型

* **输入源的数量或类型（即不同源）的更改** ：这是不允许的。
* **输入源的参数中的更改** ：是否允许这样做，以及更改的语义是否明确取决于源和查询。 以下是一些示例。
  * 允许添加/删除/修改速率限制：可以将 `spark.readStream.format("kafka").option("subscribe", "article")` 修改为 `spark.readStream.format("kafka").option("subscribe", "article").option("maxOffsetsPerTrigger", ...)`
  * 通常不允许更改已订阅的文章/文件，因为结果不可预知：不允许将 `spark.readStream.format("kafka").option("subscribe", "article")` 更改为 `spark.readStream.format("kafka").option("subscribe", "newarticle")`
* **输出接收器类型的更改** ：允许在几个特定的接收器组合之间进行更改。 这需要根据具体情况进行验证。 以下是一些示例。
  * 允许从文件接收器更改为 Kafka 接收器。 Kafka 只会看到新数据。
  * 不允许从 Kafka 接收器更改为文件接收器。
  * 允许从 Kafka 接收器更改为 foreach，反之亦然。
* **输出接收器的参数中的更改** ：是否允许这样做，以及更改的语义是否明确取决于接收器和查询。 以下是一些示例。
  * 不允许更改文件接收器的输出目录：不允许从 `sdf.writeStream.format("parquet").option("path", "/somePath")` 更改为 `sdf.writeStream.format("parquet").option("path", "/anotherPath")`
  * 允许更改输出项目：允许从 `sdf.writeStream.format("kafka").option("article", "somearticle")` 更改为 `sdf.writeStream.format("kafka").option("path", "anotherarticle")`
  * 允许更改用户定义的 foreach 接收器（即 `ForeachWriter` 代码），但更改的语义取决于代码。
* **投影/筛选器/映射类操作中的更改** ：允许某些案例。 例如： 。
  * 允许添加/删除筛选器：允许将 `sdf.selectExpr("a")` 修改为 `sdf.where(...).selectExpr("a").filter(...)`。
  * 允许对具有相同输出架构的投影进行更改：允许将 `sdf.selectExpr("stringColumn AS json").writeStream` 更改为 `sdf.select(to_json(...).as("json")).writeStream`。
  * 具有不同输出架构的投影中的更改是有条件允许的：只有当输出接收器允许架构从 `"a"` 更改为 `"b"` 时，才允许从 `sdf.selectExpr("a").writeStream` 到 `sdf.selectExpr("b").writeStream` 的更改。
* **有状态操作的更改** - 流式处理查询中的某些操作需要保留状态数据才能持续更新结果。 结构化流式处理自动为容错存储（例如，DBFS、Azure Blob 存储）的状态数据设置检查点并在重启后对其进行还原。 但是，这假设状态数据的架构在重启后保持不变。 这意味着，在两次重启之间不允许对流式处理查询的有状态操作进行任何更改（即，添加、删除或架构修改）。 下面列出了不应在两次重启之间更改其架构以确保状态恢复的有状态操作：
  * **流式处理聚合** ：例如， `sdf.groupBy("a").agg(...)`。 不允许对分组键或聚合的数量或类型进行任何更改。
  * **流式删除重复数据** ：例如， `sdf.dropDuplicates("a")`。 不允许对分组键或聚合的数量或类型进行任何更改。
  * **流间联接** ：例如 `sdf1.join(sdf2, ...)`（即两个输入都是通过 `sparkSession.readStream` 生成的）。 不允许对架构或同等联接列进行更改。 不允许对联接类型（外部或内部）进行更改。 联接条件中的其他更改被视为错误定义。
  * **任意有状态操作** ：例如 `sdf.groupByKey(...).mapGroupsWithState(...)` 或 `sdf.groupByKey(...).flatMapGroupsWithState(...)`。 不允许对用户定义状态的架构和超时类型进行任何更改。 允许在用户定义的状态映射函数中进行任何更改，但更改的语义效果取决于用户定义的逻辑。 如果你确实想要支持状态架构更改，则可以使用支持架构迁移的编码/解码方案，将复杂的状态数据结构显式编码/解码为字节。 例如，如果你将状态另存为 Avro 编码的字节，则可以在两次查询重启之间自由更改 Avro 状态架构，因为二进制状态始终会成功还原。

## <a name="configure-apache-spark-scheduler-pools-for-efficiency"></a>配置 Apache Spark 计划程序池以提高效率

默认情况下，笔记本中启动的所有查询都在同一[公平计划池](https://spark.apache.org/docs/latest/job-scheduling.html#scheduling-within-an-application)中运行。
因此，由触发器根据笔记本中的所有流式处理查询生成的作业将按照先入先出 (FIFO) 的顺序逐一运行。 这可能会导致查询中产生不必要的延迟，因为它们不能有效地共享群集资源。

若要允许所有流式处理查询并发执行作业并有效地共享群集，可以将查询设置为在单独的计划程序池中执行。 例如： 。

```scala
// Run streaming query1 in scheduler pool1
spark.sparkContext.setLocalProperty("spark.scheduler.pool", "pool1")
df.writeStream.queryName("query1").format("parquet").start(path1)

// Run streaming query2 in scheduler pool2
spark.sparkContext.setLocalProperty("spark.scheduler.pool", "pool2")
df.writeStream.queryName("query2").format("orc").start(path2)
```

> [!NOTE]
>
> 本地属性配置必须位于你启动流式处理查询时所在的笔记本单元中。

有关更多详细信息，请参阅 [Apache 公平计划程序文档](https://spark.apache.org/docs/latest/job-scheduling.html#fair-scheduler-pools)。

## <a name="optimize-performance-of-stateful-streaming-queries"></a>优化有状态流查询的性能

如果流式处理查询中存在有状态操作（例如，流式处理聚合、流式处理 dropDuplicates、流间联接、mapGroupsWithState 或 flatMapGroupsWithState），并且你希望维护数百万个处于该状态的键，则你可能会遇到与大量 JVM 垃圾回收 (GC) 暂停相关的问题，这些问题会导致微批处理时间差异过大。
出现这种情况的原因是，默认情况下，状态数据是在执行程序的 JVM 内存中维护的，大量状态对象将给 JVM 带来内存压力，导致大量的 GC 暂停。

在这种情况下，你可以选择根据 [RocksDB](https://rocksdb.org/) 使用更优化的状态管理解决方案。
此解决方案在 Databricks Runtime 中提供。 此解决方案不是在 JVM 内存中保留状态，而是使用 RocksDB 来有效地管理本机内存和本地 SSD 中的状态。 此外，对此状态所做的任何更改都会通过结构化流式处理自动保存到你提供的检查点位置，从而提供完全容错保证（与默认状态管理相同）。

可以在启动流式处理查询之前，通过在 SparkSession 中设置以下配置来启用基于 RockDB 的状态管理。

```scala
spark.conf.set(
  "spark.sql.streaming.stateStore.providerClass",
  "com.databricks.sql.streaming.state.RocksDBStateStoreProvider")
```

其他用于实现最佳性能的建议配置：

* 使用计算优化的实例作为工作器。 例如，Azure Standard_F16s 实例。
* 将无序分区的数量设置为群集中的核心数的 1-2 倍。

关于性能优势，基于 RocksDB 的状态管理可以维护的状态键是默认的 100 倍。 例如，在使用 Azure Standard_F16s 实例作为工作器的 Spark 群集中，默认状态管理可以为每个执行程序维护最多 1-2 百万个状态键，超过此数量后，JVM GC 将开始显著影响性能。 与之相对，基于 RocksDB 的状态管理可以轻松地为每个执行程序维护 1 亿个状态键，不会出现任何 GC 问题。

> [!NOTE]
>
> 无法在两次查询重启之间更改状态管理方案。 也就是说，如果使用默认管理启动了某个查询，但不使用新的检查点位置从头启动查询，则无法更改该查询。

## <a name="multiple-watermark-policy"></a>多水印策略

流式处理查询可以有多个联合或联接在一起的输入流。
对于有状态操作，每个输入流可以有不同的需要容忍的延迟数据阈值。 可以使用 `withWatermarks("eventTime", delay)` 在每个输入流上指定这些阈值。 例如，请考虑一个包含[流间联接](https://databricks.com/blog/2018/03/13/introducing-stream-stream-joins-in-apache-spark-2-3.html)的查询

```scala
val inputStream1 = ...      // delays up to 1 hour
val inputStream2 = ...      // delays up to 2 hours

inputStream1.withWatermark("eventTime1", "1 hour")
  .join(
    inputStream2.withWatermark("eventTime2", "2 hours"),
    joinCondition)
```

当执行查询时，结构化流式处理会单独跟踪每个输入流中显示的最大事件时间，根据相应的延迟计算水印，并选择一个与它们一起用于有状态操作的全局水印。 默认情况下，最小值将被选为全局水印，因为它可确保在其中一个流落后于其他流的情况下（例如，其中一个流由于上游故障而停止接收数据），数据不会因为太迟而被意外丢弃。 换句话说，全局水印将以最慢流的速度安全地移动，并且查询输出将相应地延迟。

但是，在某些情况下，你可能希望获得更快的结果，即使这意味着要从最慢的流中丢弃数据。 可以通过将 SQL 配置 `spark.sql.streaming.multipleWatermarkPolicy` 设置为 `max`（默认值为 `min`）来设置多水印策略，以选择最大值作为全局水印。
这允许全局水印以最快流的速度移动。
但副作用是较慢流中的数据会被主动丢弃。 因此，请谨慎使用此配置。

## <a name="triggers"></a>触发器

[触发器](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#triggers)定义流式数据处理的计时。 如果指定的 `trigger` 间隔太小，则系统可能会执行不必要的检查来查看新数据是否已到达。 建议指定一个定制的 `trigger` 以最大程度地降低成本，这是最佳做法。

## <a name="visualizations"></a>可视化效果

你可以使用 `display` 函数实时可视化结构化流式处理数据帧。 虽然 `trigger` 和 `checkpointLocation` 参数是可选的，但我们建议你在生产中始终指定它们，这是最佳做法。

### <a name="scala"></a>Scala

```scala
import org.apache.spark.sql.streaming.Trigger

val streaming_df = spark.readStream.format("rate").load()
display(streaming_df.groupBy().count(), trigger = Trigger.ProcessingTime("5 seconds"), checkpointLocation = "dbfs:/<checkpoint-path>")
```

### <a name="python"></a>Python

```python
streaming_df = spark.readStream.format("rate").load()
display(streaming_df.groupBy().count(), processingTime = "5 seconds", checkpointLocation = "dbfs:/<checkpoint-path>")
```

有关详细信息，请参阅[结构化流式处理数据帧](../../../notebooks/visualizations/index.md#ss-display)。