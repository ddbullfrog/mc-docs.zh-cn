---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/03/2020
title: 自适应查询执行 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用自适应查询执行。
ms.openlocfilehash: e9f439d627bdb7b4aaa33d32ad1d11384452afd7
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472886"
---
# <a name="adaptive-query-execution"></a>自适应查询执行

自适应查询执行 (AQE) 是在查询执行期间发生的查询重新优化。

运行时重新优化的推动因素是 Azure Databricks 在随机和广播交换（在 AQE 中称为查询阶段）结束时具有最新的准确统计信息。 因此，Azure Databricks 可以选择更好的物理策略、选择最佳的随机后分区大小和数目，或执行以前需要提示的优化（例如倾斜联接处理）。

这在未启用统计信息收集功能或统计信息过时的情况下会非常有用。 在静态派生的统计信息不准确的情况下（例如在复杂查询的过程中或在发生数据倾斜之后），也很有用。

## <a name="capabilities"></a>功能

在 Databricks Runtime 7.3 中，会默认启用 AQE。 它有 4 个主要功能：

* 将排序合并联接动态更改为广播哈希联接。
* 在随机交换后将分区进行动态联合（将小分区合并为大小合理的分区）。 非常小的任务具有较差的 I/O 吞吐量，并且往往会产生更多计划开销和任务设置开销。 合并小型任务可节省资源并提高群集吞吐量。
* 动态处理排序合并联接和随机哈希联接中的倾斜，方法是将倾斜的任务拆分（如果需要，还要进行复制）为大小大致相等的任务。
* 动态检测并传播空关系。

## <a name="application"></a>应用程序

AQE 适用于以下所有查询：

* 非流式处理
* 包含至少一个交换（通常在有联接、聚合或窗口时）、一个子查询或同时包含两者。

并非所有应用 AQE 的查询都需要重新优化。 重新优化产生的查询计划可能会与静态编译的查询计划不同，也可能不会。 请参阅下一节，了解如何确定 AQE 是否更改了查询的计划。

## <a name="query-plans"></a>查询计划

本部分讨论如何以不同方式检查查询计划。

### <a name="in-this-section"></a>本节内容：

* [Spark UI](#spark-ui)
* [`DataFrame.explain()`](#dataframeexplain)
* [`SQL EXPLAIN`](#sql-explain)

### <a name="spark-ui"></a>Spark UI

#### <a name="adaptivesparkplan-node"></a>`AdaptiveSparkPlan` 节点

应用了 AQE 的查询包含一个或多个 `AdaptiveSparkPlan` 节点，通常作为每个主查询或子查询的根节点。
在查询运行之前或运行时，相应的 `AdaptiveSparkPlan` 节点的 `isFinalPlan` 标志将显示为 `false`；查询执行完成后，`isFinalPlan` 标志变为 `true.`

#### <a name="evolving-plan"></a>不断发展的计划

查询计划图随着执行的进展而变化，并反映正在执行的最新计划。 已执行的节点（其中的指标可用）将不会变化，但未执行的节点可能会因重新优化而随时间的推移发生变化。

下面是查询计划图示例：

> [!div class="mx-imgBorder"]
> ![查询计划图](../../../_static/images/spark/aqe/query-plan-diagram.png)

### `DataFrame.explain()`

#### <a name="adaptivesparkplan-node"></a>`AdaptiveSparkPlan` 节点

应用了 AQE 的查询包含一个或多个 `AdaptiveSparkPlan` 节点，通常作为每个主查询或子查询的根节点。 在查询运行之前或运行时，相应的 `AdaptiveSparkPlan` 节点的 `isFinalPlan` 标志将显示为 `false`；查询执行完成后，`isFinalPlan` 标志变为 `true`。

#### <a name="current-and-initial-plan"></a>当前和初始计划

在每个 `AdaptiveSparkPlan` 节点下，将同时出现初始计划（应用任何 AQE 优化之前的计划）和当前或最终计划，具体取决于执行是否已完成。 当前计划将随执行的进展而不断变化。

#### <a name="runtime-statistics"></a>运行时统计信息

每个随机和广播阶段都包含数据统计信息。

在此阶段运行之前或运行时，统计信息是编译时估计值，标志 `isRuntime` 为 `false`，例如 `Statistics(sizeInBytes=1024.0 KiB, rowCount=4, isRuntime=false);`

阶段执行完成后，统计信息则是在运行时收集的，标志 `isRuntime` 将变成 `true`，例如 `Statistics(sizeInBytes=658.1 KiB, rowCount=2.81E+4, isRuntime=true)`

下面是一个 `DataFrame.explain` 示例：

* 执行之前

  > [!div class="mx-imgBorder"]
  > ![执行之前](../../../_static/images/spark/aqe/before-execution.png)

* 执行期间

  > [!div class="mx-imgBorder"]
  > ![执行期间](../../../_static/images/spark/aqe/during-execution.png)

* 执行之后

  > [!div class="mx-imgBorder"]
  > ![执行之后](../../../_static/images/spark/aqe/after-execution.png)

### `SQL EXPLAIN`

#### <a name="adaptivesparkplan-node"></a>`AdaptiveSparkPlan` 节点

应用 AQE 的查询包含一个或多个 AdaptiveSparkPlan 节点，通常作为每个主查询或子查询的根节点。

#### <a name="no-current-plan"></a>无当前计划

由于 `SQL EXPLAIN` 不执行查询，当前计划始终与初始计划相同，并且不反映 AQE 最终将执行什么计划。

下面是一个 SQL 说明示例：

> [!div class="mx-imgBorder"]
> ![SQL 说明](../../../_static/images/spark/aqe/sql-explain.png)

## <a name="effectiveness"></a>有效性

如果一个或多个 AQE 优化生效，查询计划将更改。 当前和最终计划与初始计划之间的差异以及当前和最终计划中的特定计划节点，反映了这些 AQE 优化的效果。

* 将排序合并联接动态更改为广播哈希联接：当前/最终计划与初始计划之间的不同物理联接节点

  > [!div class="mx-imgBorder"]
  > ![联接策略字符串](../../../_static/images/spark/aqe/join-strategy-string.png)

* 动态联合分区：带有 `Coalesced` 属性的 `CustomShuffleReader` 节点

  > [!div class="mx-imgBorder"]
  > ![CustomShuffleReader](../../../_static/images/spark/aqe/custom-shuffle-reader.png)

  > [!div class="mx-imgBorder"]
  > ![CustomShuffleReader 字符串](../../../_static/images/spark/aqe/custom-shuffle-reader-string.png)

* 动态处理倾斜联接：`isSkew` 字段为 true 的 `SortMergeJoin` 节点。

  > [!div class="mx-imgBorder"]
  > ![倾斜联接计划](../../../_static/images/spark/aqe/skew-join-plan.png)

  > [!div class="mx-imgBorder"]
  > ![倾斜联接字符串](../../../_static/images/spark/aqe/skew-join-string.png)

* 动态检测并传播空关系：部分（或整个）计划被节点 LocalTableScan 替换，其关系字段为空。

  > [!div class="mx-imgBorder"]
  > ![本地表扫描](../../../_static/images/spark/aqe/local-table-scan.png)

  > [!div class="mx-imgBorder"]
  > ![LocalTableScan 字符串](../../../_static/images/spark/aqe/local-table-scan-string.png)

## <a name="configuration"></a>配置

在此部分的属性中，将 `<prefix>` 替换为 `spark.sql.adaptive`。

### <a name="in-this-section"></a>本节内容：

* [启用和禁用自适应查询执行](#enable-and-disable-adaptive-query-execution)
* [将排序合并联接动态更改为广播哈希联接](#dynamically-change-sort-merge-join-into-broadcast-hash-join)
* [动态联合分区](#dynamically-coalesce-partitions)
* [动态处理倾斜联接](#dynamically-handle-skew-join)
* [动态检测并传播空关系](#dynamically-detect-and-propagate-empty-relations)

### <a name="enable-and-disable-adaptive-query-execution"></a>启用和禁用自适应查询执行

| 属性           | 默认 | 说明                                            |
|--------------------|---------|--------------------------------------------------------|
| `<prefix>.enabled` | 是    | 是启用还是禁用自适应查询执行。 |

### <a name="dynamically-change-sort-merge-join-into-broadcast-hash-join"></a>将排序合并联接动态更改为广播哈希联接

| 属性                              | 默认 | 说明                                                      |
|---------------------------------------|---------|------------------------------------------------------------------|
| `<prefix>.autoBroadcastJoinThreshold` | 30MB    | 运行时切换操作（切换到广播联接）的触发阈值。 |

### <a name="dynamically-coalesce-partitions"></a>动态联合分区

| 属性                                       | 默认                 | 说明                                                                                                                                              |
|------------------------------------------------|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<prefix>.coalescePartitions.enabled`          | 是                    | 是启用还是禁用分区联合。                                                                                                       |
| `<prefix>.advisoryPartitionSizeInBytes`        | 64MB                    | 联合后的目标大小。 联合后的分区大小将接近但不大于此目标大小。                                    |
| `<prefix>.coalescePartitions.minPartitionSize` | 1MB                     | 联合后的最小分区大小。 联合后的分区大小将不小于此大小。                                        |
| `<prefix>.coalescePartitions.minPartitionNum`  | 2x 群集核心数 | 联合后的最小分区数。 不建议，因为显式设置会覆盖 `<prefix>.coalescePartitions.minPartitionSize`。 |

### <a name="dynamically-handle-skew-join"></a>动态处理倾斜联接

| 属性                                            | 默认 | 说明                                                                                                          |
|-----------------------------------------------------|---------|----------------------------------------------------------------------------------------------------------------------|
| `<prefix>.skewJoin.enabled`                         | 是    | 设置 true/false 来启用/禁用倾斜联接处理。                                                                 |
| `<prefix>.skewJoin.skewedPartitionFactor`           | 5       | 一个系数，乘以分区大小中值时有助于确定分区是否倾斜。 |
| `<prefix>.skewJoin.skewedPartitionThresholdInBytes` | 256 MB   | 有助于确定分区是否倾斜的阈值。                                           |

当 `(partition size > skewedPartitionFactor * median partition size)` 和 `(partition size > skewedPartitionThresholdInBytes)` 均为 `true` 时，可认为分区倾斜。

### <a name="dynamically-detect-and-propagate-empty-relations"></a>动态检测并传播空关系

| 属性                                    | 默认 | 说明                                                      |
|---------------------------------------------|---------|------------------------------------------------------------------|
| `<prefix>.emptyRelationPropagation.enabled` | 是    | 是启用还是禁用动态空关系传播。 |

## <a name="frequently-asked-questions-faqs"></a>常见问题 (FAQ)

### <a name="in-this-section"></a>本节内容：

* [为什么 AQE 没有更改随机分区数，尽管分区联合已启用？](#why-didnt-aqe-change-the-shuffle-partition-number-despite-the-partition-coalescing-already-being-enabled)
* [为什么 AQE 未广播小型联接表？](#why-didnt-aqe-broadcast-a-small-join-table)
* [我是否仍应使用已启用 AQE 的广播联接策略提示？](#should-i-still-use-a-broadcast-join-strategy-hint-with-aqe-enabled)
* [倾斜联接提示与 AQE 倾斜联接优化之间有何区别？我应该使用哪一种？](#what-is-the-difference-between-skew-join-hint-and-aqe-skew-join-optimization-which-one-should-i-use)
* [为什么 AQE 没有自动调整我的联接顺序？](#why-didnt-aqe-adjust-my-join-ordering-automatically)
* [为什么 AQE 没有检测到数据歪斜？](#why-didnt-aqe-detect-my-data-skew)

### <a name="why-didnt-aqe-change-the-shuffle-partition-number-despite-the-partition-coalescing-already-being-enabled"></a>为什么 AQE 没有更改随机分区数，尽管分区联合已启用？

AQE 不会更改初始分区数。 建议为随机分区数设置合理的高值，并让 AQE 根据每个查询阶段的输出数据大小，联合小分区。

如果在作业中看到溢出，可以尝试执行以下操作：

* 增加随机分区数配置：`spark.sql.shuffle.partitions`
* 通过将 `<prefix>.autoOptimizeShuffle.enabled` 设置为 `true` 来启用自动随机优化

### <a name="why-didnt-aqe-broadcast-a-small-join-table"></a>为什么 AQE 未广播小型联接表？

如果预期要广播的关系的大小确实低于此阈值，但仍未广播：

* 检查联接类型。 某些联接类型不支持广播，例如，`LEFT OUTER JOIN` 的左关系无法广播。
* 也可能是因为该关系包含大量空分区，在这种情况下，大部分任务都可使用排序合并联接来快速完成，或者可使用倾斜联接处理进行优化。 如果非空分区的百分比低于 `<prefix>.nonEmptyPartitionRatioForBroadcastJoin`，AQE 会避免将此类排序合并联接更改为广播哈希联接。

### <a name="should-i-still-use-a-broadcast-join-strategy-hint-with-aqe-enabled"></a>我是否仍应使用已启用 AQE 的广播联接策略提示？

是的。 静态计划的广播联接的性能通常比由 AQE 动态计划的广播联接的性能要好，因为在同时对联接的两侧执行随机之前（实际的关系大小在这段时间内获取），AQE 可能不会切换到广播联接。 因此，如果你非常了解查询，使用广播提示仍是一个不错的选择。 AQE 将像静态优化那样遵循查询提示，但仍可应用不受提示影响的动态优化。

### <a name="what-is-the-difference-between-skew-join-hint-and-aqe-skew-join-optimization-which-one-should-i-use"></a>倾斜联接提示与 AQE 倾斜联接优化之间有何区别？ 应使用哪种方法？

建议依赖 AQE 倾斜联接处理而不使用倾斜联接提示，因为 AQE 倾斜联接完全是自动的，并且其性能通常比提示的性能要好。

### <a name="why-didnt-aqe-adjust-my-join-ordering-automatically"></a>为什么 AQE 没有自动调整我的联接顺序？

从 Databricks Runtime 7.3 开始，动态联接重新排序不再是 AQE 的一部分。

### <a name="why-didnt-aqe-detect-my-data-skew"></a>为什么 AQE 没有检测到数据歪斜？

要使 AQE 将分区检测为倾斜分区，必须满足两个有关大小的条件：

* 分区大小大于 `<prefix>.skewJoin.skewedPartitionThresholdInBytes`（默认为 256MB）
* 分区大小大于所有分区大小的中值与倾斜分区系数 `<prefix>.skewJoin.skewedPartitionFactor`（默认值为 5）的乘积

此外，对倾斜处理的支持对于某些联接类型是有限的，例如在 `LEFT OUTER JOIN` 中，只能优化左侧的倾斜。

## <a name="legacy"></a>旧的

术语“自适应执行”自 Spark 1.6 以来便已存在，但 Spark 3.0 中的新 AQE 是完全不同的一个概念。 就功能而言，Spark 1.6 只执行“动态联合分区”部分。 就技术体系结构而言，新 AQE 是一种基于运行时统计信息的动态计划和重新计划查询的框架，它支持多种优化（如本文中所述的那些优化），并可进行扩展以实现更多可能的优化。