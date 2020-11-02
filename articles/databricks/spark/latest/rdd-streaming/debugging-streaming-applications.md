---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 调试 Apache Spark 流式处理应用程序 - Azure Databricks
description: 了解如何使用 Azure Databricks 中的 UI 和日志对 Apache Spark 流式处理应用程序进行故障排除和调试。
ms.openlocfilehash: b839c17055c10edbe6e9a5c92285ac424c6815bb
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472890"
---
# <a name="debugging-apache-spark-streaming-applications"></a>调试 Apache Spark 流式处理应用程序

本指南将逐步介绍各种可用的调试选项，以便你了解 Apache Spark 流式处理应用程序的内部机制。 应注意以下三个重要地方：

* Spark UI
* 驱动程序日志
* 执行程序日志

## <a name="spark-ui"></a>Spark UI

启动流式处理作业后，Spark 和流式处理 UI 中会提供丰富的信息，方便你详细了解流式处理应用程序中发生的情况。 若要转到 Spark UI，你可单击附加的群集：

> [!div class="mx-imgBorder"]
> ![选择 Spark UI](../../../_static/images/spark/legacy-streaming/getting2-spark-ui.png)

### <a name="streaming-tab"></a>“流式处理”选项卡

进入 Spark UI 后，如果流式处理作业正在此群集中运行，则会显示“流式处理”选项卡。 如果此群集中没有正在运行的流式处理作业，此选项卡将不可见。 你可跳到[驱动程序日志](#driver-logs)了解如何检查启动流式处理作业时可能发生的异常。

在此页中需要注意的第一件事是检查流式处理应用程序是否正在从你的源接收任何输入事件。 在本例中，你可看到该作业每秒接收 1000 个事件。

<!--note: For `TextFileStream`, since files are input, the # of input events is always 0. In such cases, you can look at the **Completed Batches** section in the notebook to figure out how to find more information.-->

如果你有一个接收多个输入流的应用程序，则可单击“输入速率”链接，该链接将显示每个接收器接收的事件数。

> [!div class="mx-imgBorder"]
> ![输入速率](../../../_static/images/spark/legacy-streaming/streaming-tab.png)

### <a name="processing-time"></a>处理时间

向下滚动，找到“处理时间”关系图。 这是了解流式处理作业性能的关键图之一。 一般来说，如果你可在批处理时间的 80% 内处理每个批，那就很好。

对于此应用程序，批处理时间间隔为 2 秒。 平均处理时间为 450 毫秒，大大低于批处理时间间隔。 如果平均处理时间接近或大于批处理时间间隔，则你的流式处理应用程序将开始排队，很快就会导致积压工作 (backlog)，最终导致流式处理作业失败。

> [!div class="mx-imgBorder"]
> ![处理时间](../../../_static/images/spark/legacy-streaming/processing-time.png)

### <a name="completed-batches"></a>已完成的批处理

在页面末尾，你将看到所有已完成的批处理列表。 页面显示有关最近完成的 1000 个批处理的详细信息。 从表中，你可以获得每个批处理的事件数及其处理时间。 如果你想要详细了解某个批处理的情况，可单击该批处理链接进入“批处理详细信息”页。

> [!div class="mx-imgBorder"]
> ![已完成的批处理](../../../_static/images/spark/legacy-streaming/completed-batches.png)

### <a name="batch-details-page"></a>“批处理详细信息”页

此页面包含你想要了解的有关批处理的所有详细信息。 两个关键事项：

* 输入：它包含有关批处理输入的详细信息。 在本例中，它包含有关 Apache Kafka 主题、Spark 流式处理为此批处理读取的分区和偏移量的详细信息。 对于 TextFileStream，你将看到为此批处理读取的文件名列表。 对于从文本文件读取的流式处理应用程序，这是启动调试的最佳方式。
* 处理:你可单击作业 ID 的链接，其中包含有关在此批处理期间完成的处理的所有详细信息。

> [!div class="mx-imgBorder"]
> ![批处理详细信息](../../../_static/images/spark/legacy-streaming/batch-details-page.png)

### <a name="job-details-page"></a>“作业详细信息”页

“作业详细信息”页显示该批处理的 DStream DAG 可视化。 若要了解每个批处理操作的 DAG，这一可视化非常有用。 在本例中，可以看到批处理从 Kafka 直接流读取输入，然后执行平面映射操作，最后执行映射操作。 然后通过 updateStateByKey，使用生成的 DStream 更新全局状态。 （灰色框表示已跳过的阶段。 Spark 非常智能，如果某些阶段不需要重新计算，则会跳过。 如果该数据已设置检查点或已缓存，Spark 不会重新计算这些阶段。 在本例中，由于 `updateStateBykey`，这些阶段对应于以前的批处理的依赖项。 由于 Spark 流式处理会在 DStream 内部设置检查点，并且它会从检查点读取而不是根据以前的批处理，因此它们显示为灰色阶段。）

在页面底部，还可找到为此批处理执行的作业列表。 你可单击说明中的链接，深入了解任务级别执行。

> [!div class="mx-imgBorder"]
> ![作业详细信息](../../../_static/images/spark/legacy-streaming/job-details-page1.png)

> [!div class="mx-imgBorder"]
> ![已完成的阶段](../../../_static/images/spark/legacy-streaming/job-details-page2.png)

### <a name="task-details-page"></a>“任务详细信息”页

对于 Spark 流式处理应用程序，这是可从 Spark UI 获取的最精细调试级别。 此页面包含为此批处理执行的所有任务。 如果你正在调查流式处理应用程序的性能问题，此页会提供一些信息，例如已执行的任务数及其执行位置（在哪个执行程序上）、随机信息等。

> [!TIP]
>
> 请确保任务在群集中的多个执行程序（节点）上执行，以便在处理时具有足够的并行度。 如果你只有一个接收器，可能有时只有一个执行器来执行所有工作，尽管群集中有多个执行器。

> [!div class="mx-imgBorder"]
> ![任务详细信息](../../../_static/images/spark/legacy-streaming/task-details-page.png)

## <a name="driver-logs"></a>驱动程序日志

驱动程序日志有以下 2 个用途：

* 异常：有时，Spark UI 中可能未显示“流式处理”选项卡。 这是因为流式处理作业由于某些异常而未启动。 你可以深入驱动程序日志，查看异常的堆栈跟踪。 在某些情况下，流式处理作业可能已正常启动。 但你会发现所有批处理永远不会转到“已完成的批处理”部分。 它们可能都处于“正在处理”或“已失败”状态。 在这种情况下，借助驱动程序日志，可以很方便地了解基本问题的性质。
* 打印：作为 DStream DAG 一部分的任何 print 语句也会显示在日志中。 若要快速检查 DStream 的内容，可执行 `dstream.print()` (Scala) 或 `dstream.pprint()` (Python)。 DStream 的内容将显示在日志中。 还可执行 `dstream.foreachRDD{ print statements here }`。 它们也将显示在日志中。

  > [!NOTE]
  >
  > 只是在 DStream DAG 之外的流式处理函数中使用 print 语句时，内容将不会显示在日志中。 Spark 流式处理只生成并执行 DStream DAG。 因此，print 语句必须是该 DAG 的组成部分。

下表显示了 DStream 转换和相应日志的显示位置（如果转换有 print 语句）：

| 说明                     | 位置                     |
|---------------------------------|------------------------------|
| foreachRDD()、transform()       | 驱动程序 Stdout 日志           |
| foreachPartition()              | 执行程序的 Stdout 日志       |

若要获取驱动程序日志，可单击附加的群集。

> [!div class="mx-imgBorder"]
> ![选择驱动程序日志](../../../_static/images/spark/legacy-streaming/get2-driver-logs.png)

> [!div class="mx-imgBorder"]
> ![驱动程序日志](../../../_static/images/spark/legacy-streaming/driver-logs.png)

> [!NOTE]
>
> 对于 PySpark 流式处理，所有打印和异常不会自动显示在日志中。 当前的限制是，笔记本单元格必须处于活动状态才能显示日志。 由于流式处理作业在后台线程中运行，因此日志会丢失。 若要在运行 pyspark 流式处理应用程序时查看日志，可在笔记本的某个单元格中提供 `ssc.awaitTerminationOrTimeout(x)`。 这会使单元格等待“x”秒。 “x”秒后，该时间段内的所有打印和异常都将显示在日志中。

## <a name="executor-logs"></a>执行程序日志

如果发现某些任务行为异常，并且想要查看特定任务的日志，则执行程序日志有时会很有帮助。 在上面所示的“任务详细信息”页中，可以获取运行任务的执行程序。 获得后，即可进入“群集 UI”页，单击 # 个节点，然后单击主节点。 主节点页会列出所有工作节点。 可选择运行可疑任务的工作节点，然后转到 log4j 输出。

> [!div class="mx-imgBorder"]
> ![选择主节点](../../../_static/images/spark/legacy-streaming/clusters.png)

> [!div class="mx-imgBorder"]
> ![Spark 主节点](../../../_static/images/spark/legacy-streaming/spark-master.png)

> [!div class="mx-imgBorder"]
> ![Spark 工作节点](../../../_static/images/spark/legacy-streaming/executor-page.png)