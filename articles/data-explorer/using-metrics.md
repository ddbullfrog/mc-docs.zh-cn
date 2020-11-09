---
title: 使用指标监视 Azure 数据资源管理器的性能、运行状况和使用情况
description: 了解如何使用 Azure 数据资源管理器指标来监视群集的性能、运行状况和使用情况。
author: orspod
ms.author: v-tawe
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: how-to
origin.date: 01/19/2020
ms.date: 09/30/2020
ms.custom: contperfq1
ms.openlocfilehash: c7ddd5912ed7d13d43b376a79481657d74dc6044
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105690"
---
# <a name="monitor-azure-data-explorer-performance-health-and-usage-with-metrics"></a>使用指标监视 Azure 数据资源管理器的性能、运行状况和使用情况

Azure 数据资源管理器指标提供关于 Azure 数据资源管理器群集资源运行状况和性能的关键指标。 可以将本文中详述的指标作为独立指标，用来监视特定方案中 Azure 数据资源管理器群集的使用情况、运行状况和性能。 还可以将指标用作正常运行的 [Azure 仪表板](/azure-portal/azure-portal-dashboards)和 [Azure 警报](/azure-monitor/platform/alerts-metric-overview)的基础。

若要详细了解 Azure 指标资源管理器，请参阅[指标资源管理器](/azure-monitor/platform/metrics-getting-started)。

## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果没有，可以创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。
* 一个[群集和数据库](create-cluster-database-portal.md)。

## <a name="use-metrics-to-monitor-your-azure-data-explorer-resources"></a>使用指标来监视 Azure 数据资源管理器资源

1. 登录到 [Azure 门户](https://portal.azure.cn/)。
1. 在 Azure 数据资源管理器群集的左窗格中，搜索“指标”。
1. 选择“指标”，以打开指标窗格，然后开始对群集进行分析。
    :::image type="content" source="media/using-metrics/select-metrics.gif" alt-text="在 Azure 门户中搜索和选择指标":::

## <a name="work-in-the-metrics-pane"></a>使用指标窗格

在指标窗格中，选择要跟踪的特定指标，选择聚合数据的方式，并创建要在仪表板上查看的指标图表。

系统为 Azure 数据资源管理器群集预先选择了“资源”和“指标命名空间”选取器。   下图中的数字对应于下面带编号的列表。 这些内容可以指导你掌握在设置和查看指标时使用的不同选项。

![“指标”窗格](media/using-metrics/metrics-pane.png)

1. 若要创建指标图表，请选择 **指标** 名称和每个指标的相关 **聚合** 。 有关不同指标的详细信息，请参阅[支持的 Azure 数据资源管理器指标](#supported-azure-data-explorer-metrics)。
1. 选择“添加指标”可以查看在同一图表中绘制的多个指标。 
1. 选择“+ 新建图表”可在一个视图中查看多个图表。 
1. 使用时间选取器更改时间范围（默认：过去 24 小时）。
1. 对包含维度的指标使用 [**添加筛选器** 和 **应用拆分**](/azure-monitor/platform/metrics-getting-started#apply-dimension-filters-and-splitting)。
1. 选择“固定到仪表板”可将图表配置添加到仪表板，以便可以再次查看它。 
1. 设置 **新的警报规则** 可以使用设置的条件将指标可视化。 新的警报规则将包括图表的目标资源、指标、拆分和筛选器维度。 在[警报规则创建窗格](/azure-monitor/platform/metrics-charts#create-alert-rules)中修改这些设置。

## <a name="supported-azure-data-explorer-metrics"></a>支持的 Azure 数据资源管理器指标

Azure 数据资源管理器指标有助于深入了解资源的整体性能和使用情况，以及特定操作（如引入或查询）的相关信息。 本文中的指标已按使用类型分组。 

指标类型为： 
* [群集指标](#cluster-metrics) 
* [导出指标](#export-metrics) 
* [引入指标](#ingestion-metrics) 
* [流引入指标](#streaming-ingest-metrics)
* [查询指标](#query-metrics) 
* [具体化视图指标](#materialized-view-metrics)

有关适用于 Azure 数据资源管理器的 Azure Monitor 的指标列表（按字母顺序排列），请参阅[受支持的 Azure 数据资源管理器群集指标](/azure-monitor/platform/metrics-supported#microsoftkustoclusters)。

## <a name="cluster-metrics"></a>群集指标

群集指标跟踪群集的常规运行状况。 例如，资源和引入的使用及响应情况。

|**指标** | **单位** | **聚合** | **度量值说明** | **Dimensions** |
|---|---|---|---|---|
| 缓存利用率 | 百分比 | Avg、Max、Min | 群集当前使用的已分配缓存资源百分比。 缓存是为用户活动分配的、符合定义的缓存策略的 SSD 大小。 <br> <br> 80% 或更低的平均缓存利用率可以维持群集的正常状态。 如果平均缓存利用率高于 80%，则应对群集执行以下操作： <br> [纵向扩展](manage-cluster-vertical-scaling.md)到存储优化定价层，或者 <br> [横向扩展](manage-cluster-horizontal-scaling.md)到更多实例。 也可以调整缓存策略（减少缓存的天数）。 如果缓存利用率超过 100%，则根据缓存策略缓存的数据大小将大于群集上的缓存总大小。 | 无 |
| CPU | 百分比 | Avg、Max、Min | 群集中的计算机当前使用的已分配计算资源百分比。 <br> <br> 80% 或更低的平均 CPU 利用率可以维持群集的正常状态。 最大 CPU 利用率值为 100%，表示没有更多的计算资源可用于处理数据。 <br> 如果某个群集的性能不佳，请检查最大 CPU 利用率值，以确定特定的 CPU 是否阻塞。 | 无 |
| 引入利用率 | 百分比 | Avg、Max、Min | 用于从容量策略中分配的所有资源引入数据，以执行引入的实际资源百分比。 默认的容量策略是不超过 512 个并发的引入操作，或者不超过引入中投入的群集资源数的 75%。 <br> <br> 80% 或更低的平均引入利用率可以维持群集的正常状态。 最大的引入利用率值为 100%，表示使用整个群集引入能力，这可能会生成引入队列。 | 无 |
| 保持活动状态 | 计数 | Avg | 跟踪群集的响应度。 <br> <br> 完全可响应的群集将返回值 1，受阻或断开连接的群集将返回 0。 |
| 受限制的命令总数 | 计数 | Avg、Max、Min、Sum | 由于达到了允许的最大并发（并行）命令数，而在群集中限制（拒绝）的命令数。 | 无 |
| 盘区总数 | 计数 | Avg、Max、Min、Sum | 群集中的数据盘区总数。 <br> <br> 更改此项指标可能会更改大规模数据的结构并在群集上施加较高的负载，因为合并数据盘区是 CPU 密集型活动。 | 无 |

## <a name="export-metrics"></a>导出指标

导出指标可跟踪导出操作的常规运行状况和性能，如延迟、结果、记录数和利用率。

|**指标** | **单位** | **聚合** | **度量值说明** | **Dimensions** |
|---|---|---|---|---|
连续导出的记录数    | 计数 | Sum | 所有连续导出作业中导出的记录数。 | ContinuousExportName |
连续导出最大延迟 |    计数   | Max   | 群集中连续导出作业报告的延迟（分钟）。 | 无 |
连续导出挂起计数 | 计数 | Max   | 挂起的连续导出作业数。 这些作业已准备好运行，但可能由于容量不足而在队列中等待。 
连续导出结果    | 计数 |   计数   | 每个连续导出运行的失败/成功结果。 | ContinuousExportName |
导出利用率 |    百分比 | Max   | 已使用的导出容量占群集中总导出容量的百分比（介于 0 和 100 之间）。 | 无 |

## <a name="ingestion-metrics"></a>引入指标

引入指标可跟踪引入操作的常规运行状况和性能，如延迟、结果和数据量。

|**指标** | **单位** | **聚合** | **度量值说明** | **Dimensions** |
|---|---|---|---|---|
| 批处理 Blob 计数 | 计数 | Avg、Max、Min | 引入的已完成批处理中数据源数。 | 数据库 |
| 批处理持续时间 | 秒 | Avg、Max、Min | 引入流中批处理阶段的持续时间  | 数据库 |
| 批大小 | 字节 | Avg、Max、Min | 引入的聚合批处理中未压缩的预期数据大小。 | 数据库 |
| 已处理批处理 | 计数 | Avg、Max、Min | 引入的已完成批处理数。 `Batching Type`：批处理是否达到[批处理策略](./kusto/management/batchingpolicy.md)设置的批处理时间、数据大小或文件数限制。 | 数据库、批处理类型 |
| 发现延迟 | 秒 | Avg、Max、Min | 从数据排队开始到被数据连接发现为止的时间。 此时间未包括在 Kusto 总体引入持续时间或 KustoEventAge（引入延迟）中  | 数据库、表、数据连接类型、数据连接名称 |
| 处理的事件数（适用于事件中心/IoT 中心） | 计数 | Max、Min、Sum | 从事件中心读取的以及由群集处理的事件总数 事件划分为群集引擎拒绝的事件和接受的事件。 | EventStatus |
| 引入延迟 | 秒 | Avg、Max、Min | 引入数据的延迟，根据从群集中收到数据，到数据可供查询的时间来测得。 引入延迟周期决于引入方案。 | 无 |
| 引入结果 | 计数 | 计数 | 失败和成功的引入操作总数。 <br> <br> 使用“应用拆分”可以创建成功和失败结果桶，并分析维度（ **值** > **状态** ）。| IngestionResultDetails |
| 引入量 (MB) | 计数 | Max、Sum | 引入到群集中的数据在压缩前的总大小 (MB)。 | 数据库 |
| 阶段延迟 | 秒 | Avg、Max、Min | 某个特定组件处理这批数据的持续时间。 一批数据的所有组成部分的总阶段延迟等于这批数据的引入延迟。 | 数据库、数据连接类型、数据连接名称|

## <a name="streaming-ingest-metrics"></a>流引入指标

流引入指标跟踪流引入数据和请求速率、持续时间与结果。

|**指标** | **单位** | **聚合** | **度量值说明** | **Dimensions** |
|---|---|---|---|---|
流引入数据速率 |    Count   | RateRequestsPerSecond | 引入群集的数据总量。 | 无 |
流引入持续时间   | 毫秒  | Avg、Max、Min | 所有流引入请求的总持续时间。 | 无 |
流引入请求速率   | 计数 | Count、Avg、Max、Min、Sum | 流引入请求总数。 | 无 |
流引入结果 | 计数 | Avg   | 流引入请求总数，按结果类型列出。 | 结果 |

## <a name="query-metrics"></a>查询指标

查询性能指标跟踪查询持续时间，以及并发或受限制查询的总数。

|**指标** | **单位** | **聚合** | **度量值说明** | **Dimensions** |
|---|---|---|---|---|
| 查询持续时间 | 毫秒 | Avg、Min、Max、Sum | 收到查询结果之前所花费的总时间（不包括网络延迟）。 | QueryStatus |
| 并发查询总数 | 计数 | Avg、Max、Min、Sum | 群集中并行运行的查询数。 使用此指标可以很好地评估群集上的负载。 | 无 |
| 受限制的查询总数 | 计数 | Avg、Max、Min、Sum | 群集中受限制（被拒绝）的查询数。 允许的最大并发（并行）查询数在并发查询策略中定义。 | 无 |

## <a name="materialized-view-metrics"></a>具体化视图指标

|**指标** | **单位** | **聚合** | **度量值说明** | **Dimensions** |
|---|---|---|---|---|
|MaterializedViewHealth                    | 1、0    | 平均值     |  当视图被认为正常时，值为 1，否则为 0。 | Database、MaterializedViewName |
|MaterializedViewAgeMinutes                | 分钟数 | 平均值     | 视图的 `age` 定义为当前时间减去由视图处理的上次引入时间。 指标值是以分钟为单位的时间（值越小，视图越“正常”）。 | Database、MaterializedViewName |
|MaterializedViewResult                    | 1       | 平均值     | 指标包括 `Result` 维度，该维度指示上一个具体化循环的结果（请参见下面的可能值）。 指标值始终等于 1。 | Database、MaterializedViewName、Result |
|MaterializedViewRecordsInDelta            | 记录计数 | 平均值 | 当前在源表的未处理部分中的记录数。 有关详细信息，请参阅[具体化视图的工作原理](./kusto/management/materialized-views/materialized-view-overview.md#how-materialized-views-work)| Database、MaterializedViewName |
|MaterializedViewExtentsRebuild            | 区计数 | 平均值 | 在具体化循环中重建的区数。 | Database、MaterializedViewName|
|MaterializedViewDataLoss                  | 1       | Max    | 当未处理的源数据接近保留期时，会触发指标。 | Database、MaterializedViewName、Kind |

## <a name="next-steps"></a>后续步骤

* [教程：在 Azure 数据资源管理器中引入和查询监视数据](ingest-data-no-code.md)
* [使用诊断日志监视 Azure 数据资源管理器引入操作](using-diagnostic-logs.md)
* [快速入门：在 Azure 数据资源管理器中查询数据](web-query-data.md)
