---
title: Kusto 对添加到源的数据的更新策略 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的更新策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: bc81d3ad764eaae9e846d7cc70add9f9cbc90cfe
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105113"
---
# <a name="update-policy-overview"></a>更新策略概述

[更新策略](update-policy.md)指示 Kusto 根据对插入到源表中的数据运行的转换查询，自动在有新数据插入到源表中时将该数据追加到目标表中。

:::image type="content" source="images/updatepolicy/update-policy-overview.png" alt-text="Azure 数据资源管理器中的更新策略概述":::

例如，可以通过策略将一个表创建为另一个表的经筛选视图。 新表可以有不同的架构、保留策略，等等。 

更新策略遵循与常规引入相同的限制和最佳做法。 策略可以随群集大小进行横向扩展，在大批量引入时效率更高。

> [!NOTE]
> 源表和为其定义了更新策略的表必须位于同一数据库。
> 更新策略函数架构和目标表架构必须具有匹配的列名、类型和顺序。

## <a name="update-policys-query"></a>更新策略的查询

更新策略查询以特殊模式运行。在这种模式下，它的作用域自动限定为仅涵盖最新引入的记录，你不能在此查询中查询源表的已引入数据。 但是，事务性更新策略的“边界”中引入的数据可用于单个事务中的查询。 由于更新策略是对目标表定义的，因此将数据引入一个源表可能会导致对该数据运行多个查询。 未定义多个更新策略的执行顺序。 

### <a name="query-limitations"></a>查询限制 

* 查询可以调用存储函数，但不能包含跨数据库或跨群集查询。 
* 作为更新策略的一部分运行的查询无法读取启用了 [RestrictedViewAccess 策略](restrictedviewaccesspolicy.md)或[行级安全策略](rowlevelsecuritypolicy.md)的表。
* 在策略的 `Query` 部分中或由 `Query` 部分引用的函数中引用 `Source` 表时：
   * 不要使用表的限定名称。 请改用 `TableName`。 
   * 不要使用 `database("DatabaseName").TableName` 或 `cluster("ClusterName").database("DatabaseName").TableName`。

> [!WARNING]
> 在更新策略中定义不正确的查询会妨碍数据引入源表中。

## <a name="the-update-policy-object"></a>更新策略对象

一个表可能具有零个、一个或多个与之相关联的更新策略对象。
每一个此类对象都表示为 JSON 属性包，并定义了以下属性。

|properties |类型 |说明  |
|---------|---------|----------------|
|IsEnabled                     |`bool`  |说明是启用 (true) 还是禁用 (false) 更新策略                                                                                                                               |
|Source                        |`string`|触发更新策略调用的表的名称                                                                                                                                 |
|查询                         |`string`|用于生成更新数据的 Kusto CSL 查询                                                                                                                           |
|IsTransactional               |`bool`  |说明更新策略是否是事务性的（默认为 false）。 无法运行事务性更新策略会导致无法使用新数据对源表进行更新   |
|PropagateIngestionProperties  |`bool`  |说明数据引入源表时指定的引入属性（区标签和创建时间）是否也应该应用于派生表中的属性。                 |

> [!NOTE]
> 允许级联更新 (`TableA` → `TableB` → `TableC` → ...)。
>
> 但是，如果以循环方式定义了多个表的更新策略，则会切断更新链。 此问题是在运行时检测到的。 数据将仅向受影响表的链中的每个表引入一次。

## <a name="update-policy-commands"></a>更新策略命令

用于控制更新策略的命令包括：

* [.show table *TableName* policy update](update-policy.md#show-update-policy) 显示某个表的当前更新策略。
* [.alter table *TableName* policy update](update-policy.md#alter-update-policy) 设置某个表的当前更新策略。
* [.alter table *TableName* policy update](update-policy.md#alter-merge-table-tablename-policy-update) 追加到某个表的当前更新策略。
* [.delete table *TableName* policy update](update-policy.md#delete-table-tablename-policy-update) 删除某个表的当前更新策略。

## <a name="update-policy-is-initiated-following-ingestion"></a>更新策略在引入后启动

使用以下任意命令将数据引入或移动到定义的源表（创建区的位置）中时，更新策略会生效：

* [.ingest (pull)](../management/data-ingestion/ingest-from-storage.md)
* [.ingest (inline)](../management/data-ingestion/ingest-inline.md)
* [.set | .append | .set-or-append | .set-or-replace](../management/data-ingestion/ingest-from-query.md)
  * 将更新策略作为 `.set-or-replace` 命令的一部分进行调用时，默认行为是派生表中的数据也会被替换，就像在源表中一样。
* [.move extents](./move-extents.md)
* [.replace extents](./replace-extents.md)
  * `PropagateIngestionProperties` 命令仅在引入操作中生效。 当更新策略作为 `.move extents` 或 `.replace extents` 命令的一部分触发时，此选项无效。

## <a name="regular-ingestion-using-update-policy"></a>使用更新策略进行常规引入

当满足以下条件时，更新策略的行为类似于常规引入：

* 源表是高速率跟踪表，其中值得关注的数据的格式为自由文本列。 
* 在其上定义了更新策略的目标表仅接受特定的跟踪行。
* 该表具有结构良好的架构，该架构是对[分析运算符](../query/parseoperator.md)创建的原始自由文本数据进行转换的结果。

## <a name="zero-retention-on-source-table"></a>在源表上进行零保留

有时，引入到源表中的数据的最终目标是目标表，源表在此过程中只是一个过渡，并且你不想将原始数据保留在源表中。 请在源表的[保留策略](retentionpolicy.md)中将软删除期间定义为 0，并将更新策略设置为“事务性”。 在这种情况下： 

* 无法从源表查询源数据。 
* 引入操作期间，源数据不会永久保存到持久性存储中。 
* 操作性能会提高。 
* 用于后台整理操作的引入后资源会减少。 这些操作是在源表中的[区](../management/extents-overview.md)上完成的。

## <a name="performance-impact"></a>性能影响

更新策略会影响 Kusto 群集的性能。 更新策略会影响源表的任何引入。 引入多个数据区时，会乘以目标表的数量。 因此，对更新策略的 `Query` 部分进行优化以使其正常工作很重要。 你可以测试更新策略对引入操作的额外性能影响。 在创建或者更改策略或它在 `Query` 部分中使用的函数之前，请针对特定的现有区调用该策略。

### <a name="evaluate-resource-usage"></a>评估资源使用情况

在以下情况下，使用 [.show queries](../management/queries.md) 来评估资源使用情况（CPU、内存等的使用情况）：
* 源表名称（更新策略的 `Source` 属性）为 `MySourceTable`。
* 更新策略的 `Query` 属性调用名为 `MyFunction()` 的函数。

```kusto
.show table MySourceTable extents;
// The following line provides the extent ID for the not-yet-merged extent in the source table which has the most records
let extentId = $command_results | where MaxCreatedOn > ago(1hr) and MinCreatedOn == MaxCreatedOn | top 1 by RowCount desc | project ExtentId;
let MySourceTable = MySourceTable | where extent_id() == toscalar(extentId);
MyFunction()
```

## <a name="failures"></a>失败数

默认情况下，无法运行更新策略不会影响将数据引入源表。 但是，如果将更新策略定义为 `IsTransactional`:true，则无法运行更新策略会导致无法将数据引入到源表中。 在某些情况下，数据成功引入到源表中，但更新策略在数据引入到目标表时失败。

可以使用 [.show ingestion failures 命令](../management/ingestionfailures.md)检索更新策略时发生的故障。
 
```kusto
.show ingestion failures 
| where FailedOn > ago(1hr) and OriginatesFromUpdatePolicy == true
```

### <a name="treatment-of-failures"></a>故障处理

#### <a name="non-transactional-policy"></a>非事务性策略 

Kusto 将忽略此类失败。 任何重试均应由数据引入流程所有者负责。  

#### <a name="transactional-policy"></a>事务性策略

触发了更新的初始引入操作也将失败。 不会使用新数据修改源表和数据库。
如果引入方法为 `pull`（引入过程涉及 Kusto 的数据管理服务），则 Kusto 的数据管理服务会根据以下逻辑自动重试整个引入操作：
* 重试会一直进行，直到达到 `DataImporterMaximumRetryPeriod`（默认 = 2 天）或 `DataImporterMaximumRetryAttempts`（默认 = 10 次），以先达到的为准。
* 可以在数据管理服务的配置中更改以上两个设置。
* 退避周期从 2 分钟开始，以指数形式增长（2 -> 4 -> 8 -> 16 ... 分钟）

在其他任何情况下，任何重试均应由数据所有者负责。
