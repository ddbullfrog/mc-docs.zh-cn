---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 写入任意数据接收器 - Azure Databricks
description: 了解如何在 Azure Databricks 中将任意数据源用作流式处理数据的源和接收器。
ms.openlocfilehash: 189cb18ee25bdf3d183a68c1579165573aae7e4b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472771"
---
# <a name="write-to-arbitrary-data-sinks"></a>写入到任意数据接收器

结构化流式处理 API 提供了两种写入方式，用于将流式处理查询的输出写入尚无现有流接收器的数据源：`foreachBatch()` 和 `foreach()`。

## <a name="reuse-existing-batch-data-sources-with-foreachbatch"></a>通过 `foreachBatch()` 重复使用现有批数据源

借助 `streamingDF.writeStream.foreachBatch(...)`，你可以指定在流式处理查询每个微批处理的输出数据上执行的函数。 该函数具有两个参数：具有微批处理输出数据的 DataFrame 或 Dataset 以及微批处理的唯一 ID。 利用 `foreachBatch`，你能够：

**重复使用现有的批数据源**

许多存储系统尚无可用的流式处理接收器，但可能已具备用于批查询的数据编写器。 使用 `foreachBatch()`，你可以在每个微批处理的输出中使用批数据写入器。 以下是一些示例：

* [Cassandra Scala 示例](examples.md#foreachbatch-cassandra-example)
* [Azure Synapse Analytics Python 示例](examples.md#foreachbatch-sqldw-example)

可以通过 `foreachBatch()` 使用许多其他[批数据源](../../../data/data-sources/index.md)。

**写入多个位置**

如果要将流式处理查询的输出写入多个位置，只需多次写入输出 DataFrame/Dataset 即可。 但是，每次写入尝试都可能会导致重新计算输出数据（并可能会重新读取输入数据）。
为避免重新计算，应缓存输出 DataFrame/Dataset，将其写入多个位置，然后取消缓存。 概述如下。

```scala
streamingDF.writeStream.foreachBatch { (batchDF: DataFrame, batchId: Long) =>
  batchDF.persist()
  batchDF.write.format(...).save(...)  // location 1
  batchDF.write.format(...).save(...)  // location 2
  batchDF.unpersist()
}
```

> [!NOTE]
>
> 如果在 `batchDF` 上运行多个 Spark 作业，则流式处理查询的输入数据速率（通过 `StreamingQueryProgress` 报告并在笔记本计算机速率图中可见）可以报告为源位置生成数据的实际速率的倍数。 这是因为，每批的多个 Spark 作业可能多次读取输入数据。

**应用其他 DataFrame 操作**

流式处理 DataFrame 不支持许多 DataFrame 和 Dataset 操作，因为在这些情况下，Spark 不支持生成增量计划。
使用 `foreachBatch()`，你可以在每个微批处理输出中应用其中一些操作。 例如，可以使用 `foreachBath()` 和 SQL `MERGE INTO` 操作在更新模式下将流式处理聚合的输出写入 Delta 表。 在 [MERGE INTO](../spark-sql/language-manual/merge-into.md) 中查看更多详细信息。

> [!IMPORTANT]
>
> * `foreachBatch()` 仅提供至少一次写入保证。 但是，可以使用为函数提供的 `batchId` 来删除重复输出并获得正好一次的保证。 在这两种情况下，都必须自行考虑端到端语义。
> * `foreachBatch()` 不适用于[连续处理模式](https://databricks.com/blog/2018/03/20/low-latency-continuous-processing-mode-in-structured-streaming-in-apache-spark-2-3-0.html)，因为它基本上依赖于流式处理查询的微批处理执行。 如果在连续模式下写入数据，请改用 `foreach()`。

## <a name="write-to-any-location-using-foreach"></a>使用 `foreach()` 写入任何位置

如果不可以选择 `foreachBatch()`（例如，使用的 Databricks Runtime 版本低于 4.2，或者不存在相应的批数据编写器），则可以使用 `foreach()` 表达自定义编写器逻辑。 具体而言，可以通过以下三种方法来表达数据写入逻辑：`open()`、`process()` 和 `close()`。

### <a name="using-scala-or-java"></a>使用 Scala 或 Java

在 Scala 或 Java 中，你会扩展类 [ForeachWriter](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.ForeachWriter)：

```scala
datasetOfString.writeStream.foreach(
  new ForeachWriter[String] {

    def open(partitionId: Long, version: Long): Boolean = {
      // Open connection
    }

    def process(record: String) = {
      // Write string to connection
    }

    def close(errorOrNull: Throwable): Unit = {
      // Close the connection
    }
  }
).start()
```

### <a name="using-python"></a>使用 Python

在 Python 中，可以通过函数或对象调用 `foreach`。 该函数提供了一种简单的方法来表达处理逻辑，但在故障导致对某些输入数据进行重新处理时，不允许删除重复生成的数据。 对于这种情况，必须在对象中指定处理逻辑。

* 函数将行作为输入。

  ```python
  def processRow(row):
    // Write row to storage

  query = streamingDF.writeStream.foreach(processRow).start()
  ```

* 对象具有一个 `process` 方法以及可选的 `open` 和 `close` 方法：

  ```python
  class ForeachWriter:
    def open(self, partition_id, epoch_id):
        // Open connection. This method is optional in Python.

    def process(self, row):
        // Write row to connection. This method is not optional in Python.

    def close(self, error):
        // Close the connection. This method is optional in Python.

  query = streamingDF.writeStream.foreach(ForeachWriter()).start()
  ```

### <a name="execution-semantics"></a>执行语义

启动流式处理查询后，Spark 会通过以下方式调用函数或对象的方法：

* 此对象的单个副本负责查询中由单个任务生成的所有数据。 换句话说，一个实例负责处理以分布式方式生成的数据的一个分区。
* 此对象必须是可序列化对象，因为每个任务都将获得所提供对象的全新序列化-反序列化副本。 因此，强烈建议在调用 `open()` 方法后完成任何用于写入数据的初始化操作（例如打开连接或启动事务），这表示任务已准备好生成数据。
* 方法的生命周期如下：

  对于具有 `partition_id` 的每个分区：

  对于具有 `epoch_id` 的流式处理数据的每个批/纪元：

  调用方法 `open(partitionId, epochId)`。

  如果 `open(...)` 返回 true，则对于分区和批/纪元中的每一行，将调用方法 `process(row)`。

  在处理行时，将调用方法 `close(error)`，并出现错误（如果有）。

* 如果存在 `open()` 方法并成功返回（与返回值无关），则调用 `close()` 方法（如果存在），除非 JVM 或 Python 进程在中间出现故障。

> [!NOTE]
>
> 当故障导致对某些输入数据进行重新处理时，`open()` 方法中的 `partitionId` 和 `epochId` 可用于删除重复生成的数据。 这取决于查询的执行模式。 如果流式处理查询以微批处理模式执行，就能保证由唯一元组 `(partition_id, epoch_id)` 表示的每个分区都具有相同的数据。 因此，`(partition_id, epoch_id)` 可用于删除重复数据和/或以事务方式提交数据，并实现正好一次的保证。 但是，如果流式处理查询以连续模式执行，此保证就不成立，因此不应用于重复数据删除。