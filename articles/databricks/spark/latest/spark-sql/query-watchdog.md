---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 在交互式工作流中处理大型查询 - Azure Databricks
description: 了解如何使用查询监视器在 Azure Databricks 中限制大型查询。
ms.openlocfilehash: 41a744d1dd0217a4203fb7cbdd6a8a95a83104a4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472778"
---
# <a name="handling-large-queries-in-interactive-workflows"></a><a id="handling-large-queries-in-interactive-workflows"> </a><a id="query-watchdog"> </a>在交互式工作流中处理大型查询

交互式数据工作流的一个挑战是处理大型查询。 这包括生成过多输出行、提取许多外部分区或针对极大的数据集进行计算的查询。 这些查询的速度可能非常慢，导致群集资源饱和，并使他人难以共享同一群集。

查询监视器是一个通过检查大型查询的最常见原因并终止超过阈值的查询来防止查询垄断群集资源的过程。 本文介绍如何启用并配置查询监视器。

> [!IMPORTANT]
>
> 已为使用 UI 创建的所有用途的群集启用了查询监视器。

## <a name="example-of-a-disruptive-query"></a>中断查询示例

分析师正在实时数据仓库中执行一些即席查询。 分析师使用共享自动缩放群集，使得多个用户可以轻松地同时使用单个群集。 假设有两个表，每个表有一百万行。

```scala
import org.apache.spark.sql.functions._
spark.conf.set("spark.sql.shuffle.partitions", 10)

spark.range(1000000)
  .withColumn("join_key", lit(" "))
  .createOrReplaceTempView("table_x")
spark.range(1000000)
  .withColumn("join_key", lit(" "))
  .createOrReplaceTempView("table_y")
```

这些表的大小在 Apache Spark 中是可管理的。 但是，它们都包含一个 `join_key` 列，每行都有一个空字符串。 如果数据没有完全清除，或者某些键比其他键更普遍存在明显的数据偏斜，则可能发生这种情况。 这些空的联接键比其他任何值更普遍。

在下面的代码中，分析师将这两个表联接到它们的键上，这将产生一万亿个结果，并且所有这些都是在单个执行器（获得 `" "` 键的执行器）上生成的：

```sql
SELECT
  id, count()
FROM
  (SELECT
    x.id
  FROM
    table_x x
  JOIN
    table_y y
  on x.join_key = y.join_key)
GROUP BY id
```

该查询似乎正在运行。 但是，在不知道数据的情况下，分析师发现在执行作业的过程中仅剩下一个任务。 查询永不结束，这让分析师感到沮丧和困惑，不知道它为什么不起作用。

在这种情况下，只有一个出错联接键。 其他时候可能还有更多。

## <a name="enable-and-configure-query-watchdog"></a>启用和配置查询监视器

若要防止查询为输入行数创建过多的输出行，可以启用查询监视器并将最大输出行数配置为输入行数的倍数。  此示例使用 1000 的比率（默认值）。

```scala
spark.conf.set("spark.databricks.queryWatchdog.enabled", true)
spark.conf.set("spark.databricks.queryWatchdog.outputRatioThreshold", 1000L)
```

后一种配置声明，任何给定的任务产生的行数永远不会超过输入行数的 1000 倍。

> [!TIP]
>
> 输出比率完全可自定义。 我们建议你从较低的起点开始，看看什么阈值对你和你的团队都适用。 1,000 到 10,000 的范围是一个很好的起点。

查询监视器不仅可以防止用户为永远不会完成的作业垄断群集资源，还可以通过快速失败一个永远无法完成的查询来节省时间。 例如，以下查询将在几分钟后失败，因为它超出了比率。

```sql
SELECT
  join_key,
  sum(x.id),
  count()
FROM
  (SELECT
    x.id,
    y.join_key
  FROM
    table_x x
  JOIN
    table_y y
  on x.join_key = y.join_key)
GROUP BY join_key
```

你将看到以下内容：

> [!div class="mx-imgBorder"]
> ![查询监视器](../../../_static/images/clusters/query-watchdog-example.png)

通常，启用查询监视器并设置输出/输入阈值比率就足够了，但是你还可以选择设置两个附加属性：`spark.databricks.queryWatchdog.minTimeSecs` 和 `spark.databricks.queryWatchdog.minOutputRows`。 这些属性指定取消查询之前给定任务必须运行的最短时间，以及该查询中任务的最小输出行数。

例如，如果想要为每个任务生成大量的行，则可以将 `minTimeSecs` 设置为较高的值。 同样，如果只想在查询中的任务产生 1000 万行之后才停止查询，可以将 `spark.databricks.queryWatchdog.minOutputRows` 设置为 1000 万。 如果超过输入/输出比率，则查询成功。

```scala
spark.conf.set("spark.databricks.queryWatchdog.minTimeSecs", 10L)
spark.conf.set("spark.databricks.queryWatchdog.minOutputRows", 100000L)
```

> [!TIP]
>
> 如果在笔记本中配置查询监视器，则配置不会在群集重新启动时保留。 若要为群集的所有用户配置查询监视器，建议你使用[群集配置](../../../clusters/configure.md#spark-config)。

## <a name="detect-query-on-extremely-large-dataset"></a>在大型数据集上检测查询

另一个典型的大型查询可能会从大表/数据集中扫描大量数据。 扫描操作可能会持续很长时间，并使群集资源饱和（即使读取大 Hive 表的元数据也会花费大量时间）。 你可以设置 `maxHivePartitions` 以防止从大 Hive 表中获取太多分区。 同样，你还可以将 `maxQueryTasks` 设置为限制对超大型数据集的查询。

```scala
spark.conf.set("spark.databricks.queryWatchdog.maxHivePartitions", 20000)
spark.conf.set("spark.databricks.queryWatchdog.maxQueryTasks", 20000)
```

## <a name="when-should-you-enable-query-watchdog"></a>你应该何时启用查询监视器？

对于 SQL 分析师和数据科学家共享一个给定的群集，且管理员需要确保查询彼此“很好地运行”时，应该为即席分析群集启用查询监视器。

## <a name="when-should-you-disable-query-watchdog"></a>你应该何时禁用查询监视器？

一般来说，我们不建议急于取消在 ETL 场景中使用的查询，因为通常没有人在循环中更正错误。 我们建议你禁用除即席分析群集以外的所有查询监视器。