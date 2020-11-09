---
title: 容量策略 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的容量策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/12/2020
ms.date: 10/29/2020
ms.openlocfilehash: f9bfcbb7bd5ee44ee176951620d6fd9e3e066389
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106014"
---
# <a name="capacity-policy"></a>容量策略 

容量策略用于控制群集上数据管理操作的计算资源。

## <a name="the-capacity-policy-object"></a>容量策略对象

容量策略由以下内容组成：

* [IngestionCapacity](#ingestion-capacity)
* [ExtentsMergeCapacity](#extents-merge-capacity)
* [ExtentsPurgeRebuildCapacity](#extents-purge-rebuild-capacity)
* [ExportCapacity](#export-capacity)
* [ExtentsPartitionCapacity](#extents-partition-capacity)

## <a name="ingestion-capacity"></a>引入容量

|属性       |类型    |说明    |
|-----------------------------------|--------|-----------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |群集中并发引入操作数的最大值。               |
|CoreUtilizationCoefficient         |Double  |计算引入容量时使用的核心百分比系数。 计算得出的结果将始终由 `ClusterMaximumConcurrentOperations` 进行标准化 <br> 群集总引入容量（由 [.show capacity](../management/diagnostics.md#show-capacity) 显示）的计算方式为： <br> Minimum(`ClusterMaximumConcurrentOperations`, `Number of nodes in cluster` * Maximum(1, `Core count per node` * `CoreUtilizationCoefficient`))

> [!Note]
> 在具有三个或以上节点的群集中，管理节点不参与引入操作。 `Number of nodes in cluster` 减少 1 个。

## <a name="extents-merge-capacity"></a>盘区合并容量

|属性                           |类型    |说明                                                                                                |
|-----------------------------------|--------|-----------------------------------------------------------------------------------------------------------|
|MinimumConcurrentOperationsPerNode |long    |单个节点上并发盘区合并/重新生成操作数的最小值。 默认值为 1 |
|MaximumConcurrentOperationsPerNode |long    |单个节点上并发盘区合并/重新生成操作数的最大值。 默认值为 3 |

群集总盘区合并容量（由 [.show capacity](../management/diagnostics.md#show-capacity) 显示）的计算方式为：

`Number of nodes in cluster` x `Concurrent operations per node`

在 [`MinimumConcurrentOperationsPerNode`,`MaximumConcurrentOperationsPerNode`] 范围内，系统会自动调整 `Concurrent operations per node` 的有效值，前提是合并操作的成功率高于 90%。

> [!Note]
> 在具有三个或以上节点的群集中，管理节点不参与合并操作。 `Number of nodes in cluster` 减少 1 个。

## <a name="extents-purge-rebuild-capacity"></a>盘区清除重新生成容量

|属性                           |类型    |说明                                                                                                                           |
|-----------------------------------|--------|--------------------------------------------------------------------------------------------------------------------------------------|
|MaximumConcurrentOperationsPerNode |long    |单个节点上清除操作的并发重新生成盘区数的最大值 |

群集总盘区清除重新生成容量（由 [.show capacity](../management/diagnostics.md#show-capacity) 显示）的计算方式为：

`Number of nodes in cluster` x `MaximumConcurrentOperationsPerNode`

> [!Note]
> 在具有三个或以上节点的群集中，管理节点不参与合并操作。 `Number of nodes in cluster` 减少 1 个。

## <a name="export-capacity"></a>导出容量

|属性                           |类型    |说明                                                                                                                                                                            |
|-----------------------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |群集中并发导出操作数的最大值。                                           |
|CoreUtilizationCoefficient         |Double  |计算导出容量时使用的核心百分比系数。 计算得出的结果将始终由 `ClusterMaximumConcurrentOperations` 进行标准化。 |

群集总导出容量（由 [.show capacity](../management/diagnostics.md#show-capacity) 显示）的计算方式为：

Minimum(`ClusterMaximumConcurrentOperations`, `Number of nodes in cluster` * Maximum(1, `Core count per node` * `CoreUtilizationCoefficient`))

> [!Note]
> 在具有三个或以上节点的群集中，管理节点不参与导出操作。 `Number of nodes in cluster` 减少 1 个。

## <a name="extents-partition-capacity"></a>盘区分区容量

|属性                           |类型    |说明                                                                                         |
|-----------------------------------|--------|----------------------------------------------------------------------------------------------------|
|ClusterMinimumConcurrentOperations |long    |群集中并发盘区分区操作数的最小值。 默认值：1  |
|ClusterMaximumConcurrentOperations |long    |群集中并发盘区分区操作数的最大值。 默认值：16 |

群集总盘区分区容量（由 [.show capacity](../management/diagnostics.md#show-capacity) 显示）。

在 [`ClusterMinimumConcurrentOperations`,`ClusterMaximumConcurrentOperations`] 范围内，系统会自动调整 `Concurrent operations` 的有效值，前提是分区操作的成功率高于 90%。

## <a name="materialized-views-capacity-policy"></a>具体化视图容量策略

使用 [alter cluster policy capacity](capacity-policy.md#alter-cluster-policy-capacity) 更改容量策略。 此更改需要 `AllDatabasesAdmin` 权限。
该策略可用于更改具体化视图的并发设置。 如果在一个群集上定义了多个具体化视图，并且群集无法跟上所有视图的具体化，则可能需要进行此更改。 默认情况下，并发设置相对较低，以确保具体化不会影响群集的性能。

> [!WARNING]
> 仅当群集资源良好（低 CPU、有可用内存）时，才应增加具体化视图容量策略。 当资源有限时，增加这些值可能会导致资源耗尽，并对群集的性能产生不良影响。

具体化视图容量策略是群集[容量策略](#capacity-policy)的一部分，并且具有以下 JSON 表示形式：

<!-- csl -->
``` 
{
   "MaterializedViewsCapacity": {
    "ClusterMaximumConcurrentOperations": 1,
    "ExtentsRebuildCapacity": {
      "ClusterMaximumConcurrentOperations": 50,
      "MaximumConcurrentOperationsPerNode": 5
    }
  }
}
```

### <a name="properties"></a>属性

properties | 说明
|---|---|
|`ClusterMaximumConcurrentOperations` | 群集可并发具体化的具体化视图的最大数目。 虽然（单个视图的）具体化本身可能会运行多个并发操作，但默认情况下该值为 1。 如果在群集上定义了多个具体化视图，并且群集的资源处于良好状态，建议增加该值。 |
| `ExtentsRebuildCapacity`|  确定在具体化过程中为所有具体化视图执行的并发盘区重新生成操作的数目。 如果多个视图并发执行，则由于 `ClusterMaximumConcurrentOperation` 大于 1，它们将共享此属性定义的配额。 并发盘区重新生成操作的最大数目不会超过此值。 |

### <a name="extents-rebuild"></a>盘区重新生成

若要详细了解盘区重新生成操作，请参阅[具体化视图的工作原理](materialized-views/materialized-view-overview.md#how-materialized-views-work)。 盘区重新生成的最大数目的计算方式如下：
    
```kusto
Maximum(`ClusterMaximumConcurrentOperations`, `Number of nodes in cluster` * `MaximumConcurrentOperationsPerNode`)
```

* 默认值为总共 50 个并发重新生成，每个节点最多 5 个。
* `ExtentsRebuildCapacity` 策略仅用作上限。 系统根据当前群集的条件（内存、CPU）和重新生成操作所需的资源量的估计值，动态确定所使用的实际值。 在实际情况下，并发会远远低于容量策略中指定的值。
    * `MaterializedViewExtentsRebuild` 指标提供有关在每个具体化循环中重新生成盘区的数量的信息。 有关详细信息，请参阅[具体化视图监视](materialized-views/materialized-view-overview.md#materialized-views-monitoring)。

## <a name="defaults"></a>默认值

默认容量策略具有以下 JSON 表示形式：

```json
{
  "IngestionCapacity": {
    "ClusterMaximumConcurrentOperations": 512,
    "CoreUtilizationCoefficient": 0.75
  },
  "ExtentsMergeCapacity": {
    "MinimumConcurrentOperationsPerNode": 1,
    "MaximumConcurrentOperationsPerNode": 3
  },
  "ExtentsPurgeRebuildCapacity": {
    "MaximumConcurrentOperationsPerNode": 1
  },
  "ExportCapacity": {
    "ClusterMaximumConcurrentOperations": 100,
    "CoreUtilizationCoefficient": 0.25
  },
  "ExtentsPartitionCapacity": {
    "ClusterMinimumConcurrentOperations": 1,
    "ClusterMaximumConcurrentOperations": 16
  }
}
```

## <a name="control-commands"></a>控制命令

> [!WARNING]
> 更改容量策略之前，请咨询 Azure 数据资源管理器团队。

* 使用 [.show cluster policy capacity](capacity-policy.md#show-cluster-policy-capacity) 显示群集当前的容量策略。

* 使用 [.alter cluster policy capacity](capacity-policy.md#alter-cluster-policy-capacity) 更改群集的容量策略。

## <a name="throttling"></a>限制

Kusto 限制以下用户启动命令的并发请求数：

* 引入（包括[此处](../../ingest-data-overview.md)列出的所有命令）
   * [容量策略](#capacity-policy)中定义了限制。
* 清除
   * 当前全局固定为每个群集一个。
   * 清除重新生成容量在内部用于确定使用清除命令期间的并发重新生成操作数。 清除命令不会因此过程而被阻止/限制，但将以更快或更慢的速度运行，具体取决于清除重新生成容量。
* 导出
   * [容量策略](#capacity-policy)中定义了限制。

当群集检测到某个操作具有超过允许数量的并发操作时，它将使用 429“受限制”HTTP 代码进行响应。
回退后重试操作。
