---
title: 具体化视图 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的具体化视图。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: b9592378ac66c155d460fc13932773d49fd52d64
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106759"
---
# <a name="materialized-views-preview"></a>具体化视图（预览版）

[具体化视图](../../query/materialized-view-function.md)公开了对源表的聚合查询。 具体化视图始终返回聚合查询的最新结果（始终是全新的）。 [查询具体化视图](#materialized-views-queries)比直接对源表运行聚合的性能更高，每个查询都会执行该聚合。

> [!NOTE]
> 具体化视图有一些[限制](materialized-view-create.md#limitations-on-creating-materialized-views)，无法保证适用于所有方案。 使用此功能之前，请查看[性能注意事项](#performance-considerations)。

使用以下命令来管理具体化视图：
* [.create materialized-view](materialized-view-create.md)
* [.alter materialized-view](materialized-view-alter.md)
* [.drop materialized-view](materialized-view-drop.md)
* [.disable | .enable materialized-view](materialized-view-enable-disable.md)
* [.show materialized-views 命令](materialized-view-show-commands.md)

## <a name="why-use-materialized-views"></a>为何使用具体化视图？

通过为常用聚合的具体化视图投资资源（数据存储、后台 CPU 周期），可以获得以下优势：

* **性能提升：** 对于相同的聚合函数，查询具体化视图通常比查询源表性能更好。

* **时效性：** 具体化视图查询始终返回最新结果，而不受上次进行具体化的时间的影响。 查询组合了视图的具体化部分和源表中尚未具体化的记录（`delta` 部分），始终提供最新结果。

* **成本缩减：** 与对源表执行聚合操作相比，[查询具体化视图](#materialized-views-queries)消耗群集中的资源较少。 如果只需要聚合，则可以减少源表的保留策略。 此设置可减少源表的热缓存开销。

## <a name="materialized-views-use-cases"></a>具体化视图用例

下面是可以使用具体化视图解决的常见方案：

* 使用 [arg_max()（聚合函数）](../../query/arg-max-aggfunction.md)查询每个实体的最后一条记录。
* 使用 [any()（聚合函数）](../../query/any-aggfunction.md)消除表中的重复记录。
* 通过对原始数据计算定期统计信息来减少数据的解析。 按时间段使用各种[聚合函数](materialized-view-create.md#supported-aggregation-functions)。
    * 例如，使用 `T | summarize dcount(User) by bin(Timestamp, 1d)` 维护每天不同用户的最新快照。

有关所有用例的示例，请参阅[具体化视图 create 命令](materialized-view-create.md#examples)。

## <a name="how-materialized-views-work"></a>具体化视图的工作原理

具体化视图由两个组件组成：

* 具体化部分 - 一个Azure 数据资源管理器表，其中包含源表中已处理的聚合记录。  该表始终按聚合的分组依据组合保存单个记录。
* 增量 - 源表中尚未处理的新引入记录。

查询具体化视图会将具体化部分与增量部分组合起来，提供聚合查询的最新结果。 脱机具体化过程将增量中的新记录引入到具体化表，替换现有记录。 替换是通过重新生成保存待替换记录的区来完成的。 如果增量中的记录与具体化部分中的所有数据分片持续相交，则每个具体化循环都需要重新生成整个具体化部分，可能无法跟上引入速率。 在这种情况下，视图会变得不正常，增量会不断增长。
[监视](#materialized-views-monitoring)部分介绍了如何排查此类情况的问题。

## <a name="materialized-views-queries"></a>具体化视图查询

查询具体化视图的主要方法是按其名称进行查询，就像查询表引用一样。 查询具体化视图时，它会将视图的具体化部分与源表中尚未具体化的记录组合在一起。 查询具体化视图时，会始终根据引入到源表的所有记录返回最新结果。 有关具体化视图部件细目的详细信息，请参阅[具体化视图的工作原理](#how-materialized-views-work)。 

查询该视图的另一种方法是使用 [materialized_view() 函数](../../query/materialized-view-function.md)。 此选项支持仅查询该视图的具体化部分，同时指定用户愿意容忍的最大延迟。 此选项不保证返回最新记录，但与查询整个视图相比，此选项始终更加高效。 此函数适用于你愿意舍弃一些时效性以提高性能的方案，例如，适用于遥测仪表板。

视图可以参与跨群集或跨数据库查询，但不包括在通配符联合或搜索中。

### <a name="examples"></a>示例

1. 查询整个视图。 包括源表中的最新记录：
    
    <!-- csl -->
    ```kusto
    ViewName
    ```

1. 仅查询该视图的具体化部分，而不考虑其上次进行具体化的时间。 

    <!-- csl -->
    ```kusto
    materialized_view("ViewName")
    ```
  
## <a name="performance-considerations"></a>性能注意事项

可能会影响具体化视图运行状况的主要参与者有：

* **群集资源：** 与群集上运行的任何其他进程一样，具体化视图使用群集中的资源（CPU、内存）。 如果群集超载，将具体化视图添加到其中可能会导致群集性能下降。 可使用[群集运行状况指标](../../../using-metrics.md#cluster-metrics)监视群集的运行状况。

* **与具体化数据重叠：** 在具体化期间，自上次具体化后引入到源表中的所有新记录（增量）会被处理并具体化到视图中。 新记录与已具体化记录之间的交集越大，具体化视图的性能就越差。 如果要更新的记录数（例如，在 `arg_max` 视图中的记录数）是源表的一小部分，则具体化视图的效果最佳。 如果所有或大多数具体化视图记录需要在每个具体化循环中进行更新，则具体化视图将不能很好地执行。 请使用[区重新生成指标](../../../using-metrics.md#materialized-view-metrics)来确定这种情况。

* **引入速率：** 在具体化视图的源表中，对数据量或引入速率没有硬编码限制。 但是，具体化视图的建议引入速率不超过 1-2GB/秒。采用更高的引入速率可能仍然可以很好地执行。 性能取决于群集大小、可用资源以及与现有数据的交集量。

* **群集中的具体化视图数：** 上述注意事项适用于在群集中定义的每个单独的具体化视图。 每个视图都使用其自己的资源，许多视图会彼此争用可用资源。 对于群集中具体化视图的数量，没有硬编码限制。 但是，一般情况下，群集上的具体化视图建议不要超过 10 个。 如果在群集中定义了多个具体化视图，则可能会调整[容量策略](../capacitypolicy.md#materialized-views-capacity-policy)。

* **具体化视图定义** ：必须根据可获得最佳查询性能的查询最佳做法定义具体化视图定义。 有关详细信息，请参阅 [create 命令性能提示](materialized-view-create.md#performance-tips)。

## <a name="materialized-views-policies"></a>具体化视图策略

你可以定义实例化视图的[保留策略](../retentionpolicy.md)和[缓存策略](../cachepolicy.md)，就像任何 Azure 数据资源管理器表一样。

默认情况下，具体化视图会派生数据库保留和缓存策略。 可以使用[保留策略控制命令](../retention-policy.md)或[缓存策略控制命令](../cache-policy.md)来更改策略。
   
   * 具体化视图的保留策略与源表的保留策略无关。
   * 如果不以其他方式使用源表记录，则可以尽量减少源表的保留策略。 具体化视图仍会根据视图上设置的保留策略来存储数据。 
   * 当具体化视图处于预览模式时，建议至少允许数据存储 7 天，并将可恢复性设置为 true。 此设置可用来从错误中快速恢复以及进行诊断。
    
> [!NOTE]
> 当前不支持源表上的零保留策略。

## <a name="materialized-views-monitoring"></a>具体化视图监视

通过以下方式监视具体化视图的运行状况：

* 在 Azure 门户中监视[具体化视图指标](../../../using-metrics.md#materialized-view-metrics)：
* 监视从 [show materialized-view](materialized-view-show-commands.md#show-materialized-view) 返回的 `IsHealthy` 属性。
* 使用 [show materialized-view failures](materialized-view-show-commands.md#show-materialized-view-failures) 来检查失败。

> [!NOTE]
> 具体化永远不会跳过任何数据，即使存在恒定的失败。 该视图始终保证根据源表中的所有记录返回最新的查询快照。 恒定的失败会显著降低查询性能，但不会导致视图查询中出现不正确的结果。

### <a name="track-resource-consumption"></a>跟踪资源消耗情况

**具体化视图资源消耗：** 可以使用 [.show commands-and-queries](../commands-and-queries.md#show-commands-and-queries) 命令跟踪具体化视图具体化过程消耗的资源。 可以使用以下命令（替换 `DatabaseName` 和 `ViewName`）筛选特定视图的记录：

<!-- csl -->
```
.show commands-and-queries 
| where Database  == "DatabaseName" and ClientActivityId startswith "DN.MaterializedViews;ViewName;"
```

### <a name="troubleshooting-unhealthy-materialized-views"></a>排查不正常的具体化视图

`MaterializedViewHealth` 指标指示具体化视图是否正常。 具体化视图可能会由于以下任一或所有原因而变得不正常：
* 具体化过程失败。
* 群集没有足够的容量来按时具体化所有传入数据。 执行中不会有失败。 但是，视图仍会处于不正常状态，因为它会滞后，无法跟上引入速率。

在具体化视图变得不正常之前，其由 `MaterializedViewAgeMinutes` 指标记录的时间会逐渐增加。

### <a name="troubleshooting-examples"></a>故障排除示例

以下示例可帮助你诊断和修复不正常视图：

* **失败：** 源表已更改或删除，视图未设置为 `autoUpdateSchema`，或者源表中的更改不支持自动更新。 <br>
   **诊断：** 触发了 `MaterializedViewResult` 指标，且 `Result` 维度设置为 `SourceTableSchemaChange`/`SourceTableNotFound`。

* **失败：** 具体化过程因群集资源不足而失败，并且达到了查询限制。 <br>
  **诊断：** `MaterializedViewResult` 指标 `Result` 维度设置为 `InsufficientResources`。 Azure 数据资源管理器会尝试自动从此状态中恢复，因此，此错误可能是暂时性的。 但是，如果视图不正常，并且系统持续发出此错误，则当前群集的配置可能无法跟上引入速率，群集需要纵向或横向扩展。

* **失败：** 由于任何其他（未知）原因，具体化过程失败。 <br> 
   **诊断** ：`MaterializedViewResult` 指标的 `Result` 将是 `UnknownError`。 如果此失败频繁发生，请开具支持票证，以便 Azure 数据资源管理器团队进一步进行调查。

如果没有具体化失败，则每次成功执行时会触发 `MaterializedViewResult` 指标，且 `Result`=`Success`。 如果具体化视图滞后（`Age` 超出阈值），则它可能不正常，不管执行是否成功。 在以下情况下，可能会出现这种情形：
   * 具体化速度较慢，因为在每个具体化循环中要重新生成的区太多。 若要详细了解区重新生成为何会影响视图性能，请参阅[具体化视图的工作原理](#how-materialized-views-work)。 
   * 如果每个具体化循环都需要重新生成视图中接近 100% 的区，此视图可能无法跟上进度并会变得不正常。 每个循环中重新生成的区数在 `MaterializedViewExtentsRebuild` 指标中提供。 这种情况下，在[具体化视图容量策略](../capacitypolicy.md#materialized-views-capacity-policy)中增大区重新生成并发度可能也有所帮助。 
   * 群集中有更多具体化视图，但群集没有足够的容量来运行所有视图。 请参阅[具体化视图容量策略](../capacitypolicy.md#materialized-views-capacity-policy)来更改执行的具体化视图数量的默认设置。

## <a name="next-steps"></a>后续步骤

* [.create materialized view](materialized-view-create.md)
* [.alter materialized-view](materialized-view-alter.md)
* [具体化视图的 show 命令](materialized-view-show-commands.md)
