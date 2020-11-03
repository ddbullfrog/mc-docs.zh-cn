---
title: 连续数据导出 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的连续数据导出。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/03/2020
ms.date: 10/29/2020
ms.openlocfilehash: 0bac2dc64e3e34c4ca023c0113c2d0e3a1b37f92
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106274"
---
# <a name="continuous-data-export-overview"></a>连续数据导出概述

本文介绍了如何使用定期运行的查询，将 Kusto 中的数据连续导出到[外部表](../external-table-commands.md)。 结果会存储在外部表中，该表定义了导出的数据的目标（例如 Azure Blob 存储）和架构。 此过程确保所有记录都导出“恰好一次”，但有一些[例外](#exactly-once-export)。 

若要启用连续数据导出，请[创建一个外部表](../external-tables-azurestorage-azuredatalake.md#create-or-alter-external-table)，然后[创建连续导出定义](create-alter-continuous.md)并将其指向该外部表。 

> [!NOTE]
> 所有连续导出命令都需要[数据库管理员权限](../access-control/role-based-authorization.md)。

## <a name="continuous-export-guidelines"></a>连续导出准则

* **输出架构** ：
  * 导出查询的输出架构必须与要导出到的外部表的架构匹配。 
* **频率** ：
  * 连续导出将根据在 `intervalBetweenRuns` 属性中为其配置的时间段运行。 此间隔的建议值至少为几分钟，具体取决于你愿意接受的延迟时间。 如果引入速率较高，则时间间隔最小可为一分钟。
* **分布** ：
  * 连续导出中的默认分布是 `per_node`，其中的所有节点都同时导出。 
  * 可以在连续导出的 create 命令的属性中重写此设置。 使用 `per_shard` 分布可提高并发性。
    > [!NOTE]
    > 此分布会增加存储帐户的负载，并有可能达到限制上限。 
  * 使用 `single`（或 `distributed`=`false`）来完全禁用分发。 此设置可能会显著降低连续导出过程的速度，并影响每个连续导出迭代中创建的文件数。 
* **文件数目** ：
  * 每个连续导出迭代中导出的文件数取决于外部表的分区情况。 有关详细信息，请参阅[导出到外部表命令](export-data-to-an-external-table.md#number-of-files)。 每个连续导出迭代始终会写入到新文件中，永远不会附加到现有文件中。 因此，导出的文件的数目也取决于连续导出的运行频率。 频率参数为 `intervalBetweenRuns`。
* **位置** ：
  * 为了获得最佳性能，Azure 数据资源管理器群集和存储帐户应位于同一 Azure 区域中。
  * 如果导出的数据量很大，建议为外部表配置多个存储帐户，以免出现存储限制。 请参阅[将数据导出到存储](export-data-to-storage.md#known-issues)。

## <a name="exactly-once-export"></a>导出恰好一次

为了保证导出“恰好一次”，连续导出使用[数据库游标](../databasecursor.md)。 必须在所有已在查询中引用且应当在导出中处理“恰好一次”的表上启用 [IngestionTime 策略](../ingestiontime-policy.md)。 此策略默认在所有新创建的表上启用。

导出“恰好一次”的保证仅适用于[显示导出的项目命令](show-continuous-artifacts.md)中报告的文件。 连续导出并不保证仅将每个记录向外部表中写入一次。 如果在开始导出后发生故障，并且某些项目已被写入到外部表，则外部表可能包含重复项。 如果写入操作在完成之前被中止，则外部表可能包含已损坏的文件。 在这种情况下，不会从外部表中删除这些项目，但也不会在[显示导出的项目命令](show-continuous-artifacts.md)中报告这些项目。 通过 `show exported artifacts command` 使用导出的文件可保证无重复且无损坏。

## <a name="export-to-fact-and-dimension-tables"></a>导出到事实数据表和维度表

默认情况下，会假定导出查询中引用的所有表都是[事实数据表](../../concepts/fact-and-dimension-tables.md)。 因此，它们的作用域限定于数据库游标。 此语法显式声明哪些表具有作用域（事实数据表）以及哪些表没有作用域（维度表）。 有关详细信息，请参阅[创建命令](create-alter-continuous.md)中的 `over` 参数。

导出查询仅包含自上一次执行导出后联接的记录。 导出查询可能包含[维度表](../../concepts/fact-and-dimension-tables.md)，维度表的所有记录都包括在所有导出查询中。  在连续导出中，当在事实数据表与维度表之间使用联接时，请记住，事实数据表中的记录仅处理一次。 如果在维度表中的记录缺少某些键的情况下运行导出，则相应键的记录会丢失，或者会在导出的文件中包含维度列的 null 值。 是返回缺少的记录还是返回 null 记录取决于查询使用的是内部联接还是外部联接。 如果对于匹配的记录，事实数据表和维度表是在同一时间引入的，则在这种情况下，连续导出定义中的 `forcedLatency` 属性可能很有用。

> [!NOTE]
> 不支持连续导出纯维度表。 导出查询必须包含至少一个事实数据表。

## <a name="exporting-historical-data"></a>导出历史数据

连续导出只从其创建时间点开始导出数据。 在该时间之前引入的记录应该使用非连续[导出命令](export-data-to-an-external-table.md)单独导出。 

历史数据可能太大，无法在单个导出命令中导出。 如果需要，请将查询分成几个较小的批。 

为了避免与连续导出所导出的数据重复，请使用由[显示连续导出命令](show-continuous-export.md)返回的 `StartCursor`，仅导出 `where cursor_before_or_at` 该游标值的记录。 例如：

```kusto
.show continuous-export MyExport | project StartCursor
```

| StartCursor        |
|--------------------|
| 636751928823156645 |

后面是： 

```kusto
.export async to table ExternalBlob
<| T | where cursor_before_or_at("636751928823156645")
```

## <a name="resource-consumption"></a>资源消耗

* 连续导出对群集的影响取决于连续导出正在运行的查询。 大多数资源（例如 CPU 和内存）都由查询执行操作使用。 
* 可以并发运行的导出操作的数量受限于群集的数据导出容量。 有关详细信息，请参阅[限制](../../management/capacitypolicy.md#throttling)。 如果群集没有足够的容量来处理所有连续导出，某些导出将会滞后启动。
* 可以使用 [show commands-and-queries 命令](../commands-and-queries.md)来估计资源消耗情况。 
  * 根据 `| where ClientActivityId startswith "RunContinuousExports"` 进行筛选，查看与连续导出关联的命令和查询。

## <a name="limitations"></a>限制

* 连续导出对于通过流式引入引入的数据不起作用。 
* 无法在启用了[行级安全策略](../../management/rowlevelsecuritypolicy.md)的表上配置连续导出。
* 其[连接字符串](../../api/connection-strings/storage.md)中包含 `impersonate` 的外部表不支持连续导出。
* 连续导出不支持跨数据库和跨群集调用。
* 连续导出未设计用于将数据从 Azure 数据资源管理器连续流式传输出去。 连续导出以分布式模式运行，所有节点并发导出。 如果每个运行所查询的数据范围很小，则连续导出的输出将是很多小项目。 项目数量取决于群集中的节点数。

<!-- * If the artifacts used by continuous export are intended to trigger Event Grid notifications, see the [known issues section in the Event Grid documentation](../../../ingest-data-event-grid-overview.md#known-event-grid-issues). -->
 