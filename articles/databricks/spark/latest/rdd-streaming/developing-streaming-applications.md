---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 开发流式处理应用程序的最佳做法 - Azure Databricks
description: 了解有关在 Azure Databricks 中开发 Apache Spark Streaming 应用程序的最佳做法。
ms.openlocfilehash: 6879f803188d9e9070fe4858c1381223df9d2dd5
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472888"
---
# <a name="best-practices-for-developing-streaming-applications"></a>开发流式处理应用程序的最佳做法

本文为在 Azure Databricks 笔记本中开发生产质量 Apache Spark Streaming 应用程序提供了一些指针。 本文重点介绍在开发这些应用程序时通常会遇到的问题，并举例说明最佳做法。

## <a name="serialization-issues"></a>序列化问题

笔记本提供了一个不错的开发环境。 你可以像运行单个单元一样快速地循环访问代码。 开发周期结束后，可以轻松地将笔记本传输到生产工作负载。
但对于 Spark Streaming 应用程序而言，此开发过程可能会变得很繁琐。 以下部分介绍了有关如何克服在开发 Spark Streaming 应用程序时遇到的最常见问题之一的技巧：`NotSerializableException`。

开发 Spark 应用程序时，通常会遇到如下所示的堆栈跟踪：

```
org.apache.spark.SparkException: Task not serializable
  at org.apache.spark.util.ClosureCleaner$.ensureSerializable(ClosureCleaner.scala:304)
  at org.apache.spark.util.ClosureCleaner$.org$apache$spark$util$ClosureCleaner$$clean(ClosureCleaner.scala:294)
  at org.apache.spark.util.ClosureCleaner$.clean(ClosureCleaner.scala:122)
  at org.apache.spark.SparkContext.clean(SparkContext.scala:2058)
  ...
Caused by: java.io.NotSerializableException
```

很难解释在哪个闭包中捕获了什么，这个闭包要求哪个类是可序列化的（或者理解前面的句子！）。 以下是一些有关避免 `NotSerializableException` 的最佳方法的准则：

* 尽可能多地声明 `Object` 中的函数
* 如果需要在闭包内（例如，在 foreachRDD 中）使用 `SparkContext` 或 `SQLContext`，请改用 `SparkContext.get()` 和 `SQLContext.getActiveOrCreate()`
* 重新定义为函数内部类构造函数提供的变量

有关具体的实现示例，请参阅本文末尾的[示例 Spark Streaming 应用程序](#example)。

Spark 使用 [SerializationDebugger](https://spark.apache.org/docs/latest/api/java/org/apache/spark/serializer/SerializationDebugger.html) 作为默认调试器来检测序列化问题，但有时可能会遇到 `SerializationDebugger: java.lang.StackOverflowError` 错误。 也可以通过启用 JVM 的 `sun.io.serialization.extendedDebugInfo` 标志来关闭它。 创建群集时，在 [Spark 配置](../../../clusters/configure.md#spark-config) 中设置以下属性：

```scala
spark.driver.extraJavaOptions -Dsun.io.serialization.extendedDebugInfo=true
spark.executor.extraJavaOptions -Dsun.io.serialization.extendedDebugInfo=true
```

## <a name="checkpointing-recovery-issues"></a>检查点恢复问题

开发生产质量 Spark Streaming 应用程序时，有一个突出的要求，即容错。 作为长时间运行的应用程序，如果发生故障，应用程序必须能够从中断的地方恢复。

检查点是使 Spark Streaming 容错的机制之一。 启用检查点后，会发生两种情况：

* 所有 DStream 转换的有向无环图 (DAG) 都将序列化并存储在可靠的存储中。
* 如果正在运行有状态操作（例如 `mapWithState`、`updateStateByKey`），则在处理每个批处理后，状态将序列化并存储在可靠的存储中。

可以通过提供检查点目录在流式处理应用程序中启用检查点：

```scala
val checkpointDir = ...

def creatingFunc(): StreamingContext = {
   val newSsc = ...                      // create and setup a new StreamingContext

   newSsc.checkpoint(checkpointDir)      // enable checkpointing

   ...
}

// Recreate context from checkpoints info in checkpointDir, or create a new one by calling the function.
val ssc = StreamingContext.getOrCreate(checkpointDir, creatingFunc _)
```

在 DStream 操作中使用的任何内容（例如 `transform`、`foreachRDD` 等）都必须是可序列化的，以便 Spark Streaming 可以将其存储以实现驱动程序容错。 序列化并存储 DAG 后，只要没有更改代码，就可以在单独的群集上重启应用程序，并且应用程序仍然可以运行。

对于使用 JAR 计划的工作，这是个好消息。 你可以为 Spark Streaming 作业计划重试，在出现故障时，作业将重启，你可以从中断的位置重新开始。

但是使用笔记本时会有一条注意事项 。 如果笔记本作业重启，则可能会遇到如下所示的令人讨厌的堆栈跟踪：

```
org.apache.spark.SparkException: Failed to read checkpoint from directory ...
  at org.apache.spark.streaming.CheckpointReader$.read(Checkpoint.scala:367)
    at org.apache.spark.streaming.StreamingContext$.getOrCreate(StreamingContext.scala:862)
    at org.apache.spark.streaming.StreamingContext$$anonfun$getActiveOrCreate$1.apply(StreamingContext.scala:838)
    at org.apache.spark.streaming.StreamingContext$$anonfun$getActiveOrCreate$1.apply(StreamingContext.scala:838)
    at scala.Option.getOrElse(Option.scala:120)
    at org.apache.spark.streaming.StreamingContext$.getActiveOrCreate(StreamingContext.scala:838)
Caused by: java.io.IOException: java.lang.ClassNotFoundException: line8c9ff88e00d34452b053d892b6d2a6d720.$read$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$anonfun$4
```

Spark 无法找到的繁琐的类是什么？

每个笔记本都在后台使用 REPL。 这会使定义为包装在闭包内的所有类和函数产生繁琐、晦涩的类名。 例如，考虑下面的代码。

```scala
dstream.map { x => (x, 1) }
```

内联函数 `x => (x, 1)` 由 REPL 编译成一个匿名类和名为 `$read$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$anonfun$` 的函数。

这些匿名类将序列化并保存到检查点文件中。 重启群集后，新的 REPL 将具有不同的 ID，从而导致生成的类名也不同。 因此，无法对检查点文件中的类进行反序列化以恢复 `StreamingContext`，这将导致 `ClassNotFoundException`。

### <a name="solution"></a>解决方案

解决此问题的方法是使用[包单元](../../../notebooks/package-cells.md)：

```scala
package x.y.z
```

包单元内的任何内容均使用给定的包命名空间进行编译，而不是使用繁琐的匿名命名空间。 若要解决此问题，可以将函数移动到同一笔记本中的其他“包”单元。 它将包含以下内容。

```scala
package example  // whatever package name you want

object MyFunctions {
  def mapFunc(x: String) = (x, 1)
}
```

这将生成具有完全限定名称 `example.MyFunctions` 的类。 然后，可以将之前的代码更改为以下代码。

```scala
dstream.map(example.MyFunctions.mapFunc)
```

这将允许从检查点文件进行恢复。 因此，为了从驱动程序故障中正确地恢复，你应该：

* 将应用程序中使用的所有类移到包单元中
* 定义包单元内的对象中的所有函数

## <a name="example-spark-streaming-application"></a><a id="example"> </a><a id="example-spark-streaming-application"> </a>Spark Streaming 应用程序示例

下面是一个简单的示例，以一种可靠、容错的方式将 1 添加到整数流中，然后将其可视化。 我们将使用本文中所有的提示和技巧来开发和调试应用程序。

可以使用以下接收器生成数据：

```scala
package com.databricks.example

import scala.util.Random

import org.apache.spark._
import org.apache.spark.storage._
import org.apache.spark.streaming._
import org.apache.spark.streaming.receiver._

/** This is a dummy receiver that generates data. */
class DummySource extends Receiver[(Int, Long)](StorageLevel.MEMORY_AND_DISK_2) {

  /** Start the thread that receives data over a connection */
  def onStart() {
    new Thread("Dummy Source") { override def run() { receive() } }.start()
  }

  def onStop() {  }

  /** Periodically generate a random number from 0 to 9, and the timestamp */
  private def receive() {
    while(!isStopped()) {
      store(Iterator((Random.nextInt(10), System.currentTimeMillis)))
      Thread.sleep(1000)
    }
  }
}
```

### <a name="bad-stream-example"></a>错误的流示例

以下是设置流的错误方法。

```scala
package com.databricks.example

import org.apache.spark._
import org.apache.spark.storage._
import org.apache.spark.streaming._
import org.apache.spark.streaming.receiver._
import org.apache.spark.sql._
import org.apache.spark.sql.functions._
import com.databricks.example._

class AddOneStream(sc: SparkContext, sqlContext: SQLContext, cpDir: String) {

  // This will cause a NotSerializableException while checkpointing,
  // as it will unintentionally bring sqlContext object and associated non-serializable REPL objects in scope
  import sqlContext.implicits._

  def creatingFunc(): StreamingContext = {

    val batchInterval = Seconds(1)
    val ssc = new StreamingContext(sc, batchInterval)
    ssc.checkpoint(cpDir)

    val stream = ssc.receiverStream(new DummySource())

    // This function will cause a NotSerializableException on class AddOneStream while running the job,
    // as they will require `AddOneStream` to be serialized in order to be used inside DStream functions.
    def addOne(value: (Int, Long)): (Int, Long) = {
      (value._1 + 1, value._2)
    }

    stream.map(addOne).window(Minutes(1)).foreachRDD { rdd =>

      // This will cause a NotSerializableException while checkpointing,
      // as it will unintentionally bring sqlContext object and associated non-serializable REPL objects in scope
      sqlContext.createDataFrame(rdd).toDF("value", "time")
        .withColumn("date", from_unixtime($"time" / 1000))
        .createOrReplaceTempView("demo_numbers")
    }

    ssc
  }
}
```

```scala
import com.databricks.example._
import org.apache.spark.streaming._
val cpDir = "dbfs:/home/examples/serialization"

val addOneStream = new AddOneStream(sc, sqlContext, cpDir)
val ssc = StreamingContext.getActiveOrCreate(cpDir, addOneStream.creatingFunc _)
ssc.start()
```

由于以下原因，上一定义不起作用：

* `addOne` 函数要求对 `AddOneStream` 进行序列化，但它不是 `Serializable` 的。
* 我们尝试过在 `foreachRDD` 中对 `sqlContext` 进行序列化。
* `import sqlContext.implicits._` 是在类中定义的，但在 `foreachRDD` 中进行了访问，因此也需要对 `AddOneStream` 进行序列化。

### <a name="good-stream-example"></a>正确的流示例

所有这些问题都可以解决，如下所示：

```scala
package com.databricks.example2

import org.apache.spark._
import org.apache.spark.storage._
import org.apache.spark.streaming._
import org.apache.spark.streaming.receiver._
import org.apache.spark.sql._
import org.apache.spark.sql.functions._
import com.databricks.example._

class AddOneStream(sc: SparkContext, sqlContext: SQLContext, cpDir: String) {

  // import the functions defined in the object.
  import AddOneStream._

  def creatingFunc(): StreamingContext = {

    val batchInterval = Seconds(1)
    val ssc = new StreamingContext(sc, batchInterval)

    // Set the active SQLContext so that we can access it statically within the foreachRDD
    SQLContext.setActive(sqlContext)

    ssc.checkpoint(cpDir)

    val stream = ssc.receiverStream(new DummySource())

    stream.map(addOne).window(Minutes(1)).foreachRDD { rdd =>

      // Access the SQLContext using getOrCreate
      val _sqlContext = SQLContext.getOrCreate(rdd.sparkContext)
      _sqlContext.createDataFrame(rdd).toDF("value", "time")
        .withColumn("date", from_unixtime(col("time") / 1000))
        // we could have imported _sqlContext.implicits._ and used $"time"
        .createOrReplaceTempView("demo_numbers")
    }

    ssc
  }
}

object AddOneStream {
  def addOne(value: (Int, Long)): (Int, Long) = {
    (value._1 + 1, value._2)
  }
}
```

```scala
import com.databricks.example2._
import org.apache.spark.streaming._

val addOneStream = new AddOneStream(sc, sqlContext, cpDir)
val ssc = StreamingContext.getActiveOrCreate(cpDir, addOneStream.creatingFunc _)
ssc.start()
```

请放心运行以下单元，你将拥有一个可用于生产的流式处理应用程序！

```sql
select * from demo_numbers
```