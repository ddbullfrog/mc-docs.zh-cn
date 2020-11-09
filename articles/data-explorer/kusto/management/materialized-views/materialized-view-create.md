---
title: 创建具体化视图 - Azure 数据资源管理器
description: 本文介绍了如何在 Azure 数据资源管理器中创建具体化视图。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: dd3cf9016cb4d9daafb10a0e7f973ed65c906d54
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106575"
---
# <a name="create-materialized-view"></a>.create materialized-view

[具体化视图](materialized-view-overview.md)是对源表的聚合查询，表示单个 summarize 语句。

创建具体化视图有两种可能的方法，由命令中的 backfill 选项指示：

 * **基于源表中的现有记录创建：** 
      * 创建过程可能需要很长时间才能完成，具体取决于源表中的记录数。 在完成之前，视图不可用于查询。
      * 使用此选项时，create 命令必须是 `async` 的，可以使用 [.show operations](../operations.md#show-operations) 命令来监视执行。

    * 可以使用 [.cancel operation](#cancel-materialized-view-creation) 命令取消回填过程。

      > [!IMPORTANT]
      > * 对于冷缓存中的数据，不支持使用回填选项。 如有必要，请增加用于创建视图的热缓存期限。 这可能需要横向扩展。    
      > * 对于大型源表，使用回填选项可能需要很长时间才能完成。 如果此过程在运行时暂时失败，将不会自动重试，需要重新执行 create 命令。
    
* **从头开始创建具体化视图：** 
    * 具体化视图创建后为空，只会包括在创建视图后引入的记录。 此类型的创建操作会立即返回，不需要 `async`，而视图将立即可用于查询。

创建操作需要[数据库管理员](../access-control/role-based-authorization.md)权限。 具体化视图的创建者将成为它的管理员。

## <a name="syntax"></a>语法

`.create` [`async`] `materialized-view` <br>
[ `with` `(`*PropertyName* `=` *PropertyValue*`,`...`)`] <br>
*ViewName* `on table` *SourceTableName* <br>
`{`<br>&nbsp;&nbsp;&nbsp;&nbsp;*Query*<br>`}`

## <a name="arguments"></a>参数

|参数|类型|说明
|----------------|-------|---|
|视图名|字符串|具体化视图名称。 视图名称不能与同一数据库中的表名或函数名冲突，并且必须遵循[标识符命名规则](../../query/schema-entities/entity-names.md#identifier-naming-rules)。 |
|SourceTableName|字符串|定义视图时依据的源表的名称。|
|查询|String|具体化视图查询。 有关详细信息，请参阅 [query](#query-argument)。|

### <a name="query-argument"></a>Query 参数

具体化视图参数中使用的查询受以下规则限制：

* query 参数应引用作为具体化视图的源的单个事实数据表，包含单个 summarize 运算符，以及一个或多个按一个或多个分组依据表达式聚合的聚合函数。 summarize 运算符必须始终是查询中的最后一个运算符。

* 视图可以是 `arg_max`/`arg_min`/`any` 视图（这些函数可在同一视图中一起使用）或任何其他受支持的函数，但这两者不能同时用于同一具体化视图中。 
    例如，`SourceTable | summarize arg_max(Timestamp, *), count() by Id` 不受支持。 

* 查询不应包含依赖于 `now()` 或 `ingestion_time()` 的任何运算符。 例如，查询不应具有 `where Timestamp > ago(5d)`。 使用 `arg_max`/`arg_min`/`any` 聚合的具体化视图不能包含任何其他受支持的聚合函数。 对具体化视图使用保留策略来限制视图所覆盖的时间段。

* 具体化视图定义中不支持复合聚合。 例如，请将具体化视图定义为 `SourceTable | summarize a=sum(Column1), b=sum(Column2) by Id`（而不是定义为以下视图：`SourceTable | summarize Result=sum(Column1)/sum(Column2) by Id`）。 在进行视图查询时，运行 - `ViewName | project Id, Result=a/b`。 可以将视图的必需输出（包括计算列 (`a/b`)）封装在[存储函数](../../query/functions/user-defined-functions.md)中。 请访问存储函数，而不是直接访问具体化视图。

* 不支持跨群集/跨数据库查询。

* 不支持引用 [external_table()](../../query/externaltablefunction.md) 和 [externaldata](../../query/externaldata-operator.md)。

* 除了视图的源表之外，它还可以引用一个或多个 [`dimension tables`](../../concepts/fact-and-dimension-tables.md)。 必须在视图属性中显式调用维度表。 必须了解与维度表联接时的行为：

    * 视图的源表（事实数据表）中的记录仅具体化一次。 事实数据表与维度表之间的不同引入延迟可能会影响视图结果。

    * **示例** ：视图定义包含一个与维度表的内联。 在具体化时，维度记录未完全引入，但已引入到事实数据表。 此记录会从视图中删除，不会再次重新处理。 

        同样，如果联接是外部联接，则会处理事实数据表中的记录，并将其添加到视图，其中维度表列具有 null 值。 已添加到视图的记录（具有 null 值）不会再次处理。 它们在维度表的列中的值将保留为 null。

## <a name="properties"></a>属性

`with(propertyName=propertyValue)` 子句中支持以下各项。 所有属性都是可选的。

|properties|类型|说明 |
|----------------|-------|---|
|backfill|bool|根据 SourceTable 中当前存在的所有记录创建视图 (`true`)，还是从现在开始创建视图 (`false`)。 默认值为 `false`。| 
|effectiveDateTime|datetime| 如果与 `backfill=true` 一起指定，则创建操作仅会回填在该日期/时间之后引入的记录。 还必须将回填设置为 true。 需要一个日期/时间文本，例如 `effectiveDateTime=datetime(2019-05-01)`|
|dimensionTables|Array|以逗号分隔的、视图中的维度表的列表。 请参阅 [Query 参数](#query-argument)
|autoUpdateSchema|bool|是否根据源表更改自动更新视图。 默认值为 `false`。 此选项仅对 `arg_max(Timestamp, *)` / `arg_min(Timestamp, *)` / `any(*)` 类型的视图（仅当 columns 参数为 `*` 时）有效。 如果将此选项设置为 true，则对源表所做的更改会自动反映在具体化视图中。
|文件夹|string|具体化视图的文件夹。|
|docString|string|记录具体化视图的字符串|

> [!WARNING]
> * 删除源表中的列时，使用 `autoUpdateSchema` 可能会导致不可恢复的数据丢失。 
> * 如果对源表所做的更改导致具体化视图的架构更改，并且 `autoUpdateSchema` 为 false，则该视图会被自动禁用。 
>    * 当使用 `arg_max(Timestamp, *)` 并向源表中添加列时，此错误很常见。 
>    * 可以通过将视图查询定义为 `arg_max(Timestamp, Column1, Column2, ...)` 或使用 `autoUpdateSchema` 选项来避免此错误。
> * 如果出于上述原因禁用了视图，则可以在解决问题后使用 [enable materialized view](materialized-view-enable-disable.md) 命令来重新启用它。
>

## <a name="examples"></a>示例

1. 创建一个空的 arg_max 视图，它只会具体化从现在开始引入的记录：

    <!-- csl -->
    ```
    .create materialized-view ArgMax on table T
    {
        T | summarize arg_max(Timestamp, *) by User
    }
    ```
    
1. 使用 `async` 通过回填选项为每日聚合创建具体化视图：

    <!-- csl -->
    ```
    .create async materialized-view with (backfill=true, docString="Customer telemetry") CustomerUsage on table T
    {
        T 
        | extend Day = bin(Timestamp, 1d)
        | summarize count(), dcount(User), max(Duration) by Customer, Day 
    } 
    ```
    
1. 使用回填和 `effectiveDateTime` 创建具体化视图。 视图是仅基于此日期/时间之后的记录创建的：

    <!-- csl -->
    ```
    .create async materialized-view with (backfill=true, effectiveDateTime=datetime(2019-01-01)) CustomerUsage on table T 
    {
        T 
        | extend Day = bin(Timestamp, 1d)
        | summarize count(), dcount(User), max(Duration) by Customer, Day
    } 
    ```
1. 基于 EventId 列对源表执行重复数据删除操作的具体化视图：

    <!-- csl -->
    ```
    .create materialized-view DedupedT on table T
    {
        T
        | summarize any(*) by EventId
    }
    ```

1. 定义可以在 `summarize` 语句之前包含其他运算符，只要 `summarize` 是最后一个运算符即可：

    <!-- csl -->
    ```
    .create materialized-view CustomerUsage on table T 
    {
        T 
        | where Customer in ("Customer1", "Customer2", "CustomerN")
        | parse Url with "https://contoso.com/" Api "/" *
        | extend Month = startofmonth(Timestamp)
        | summarize count(), dcount(User), max(Duration) by Customer, Api, Month
    }
    ```

1. 与维度表联接的具体化视图：

    <!-- csl -->
    ```
    .create materialized-view EnrichedArgMax on table T with (dimensionTables = ['DimUsers'])
    {
        T
        | lookup DimUsers on User  
        | summarize arg_max(Timestamp, *) by User 
    }
    
    .create materialized-view EnrichedArgMax on table T with (dimensionTables = ['DimUsers'])
    {
        DimUsers | project User, Age, Address
        | join kind=rightouter hint.strategy=broadcast T on User
        | summarize arg_max(Timestamp, *) by User 
    }
    ```
    

## <a name="supported-aggregation-functions"></a>支持的聚合函数

支持以下聚合函数：

* [`count`](../../query/count-aggfunction.md)
* [`countif`](../../query/countif-aggfunction.md)
* [`dcount`](../../query/dcount-aggfunction.md)
* [`dcountif`](../../query/dcountif-aggfunction.md)
* [`min`](../../query/min-aggfunction.md)
* [`max`](../../query/max-aggfunction.md)
* [`avg`](../../query/avg-aggfunction.md)
* [`avgif`](../../query/avgif-aggfunction.md)
* [`sum`](../../query/sum-aggfunction.md)
* [`arg_max`](../../query/arg-max-aggfunction.md)
* [`arg_min`](../../query/arg-min-aggfunction.md)
* [`any`](../../query/any-aggfunction.md)
* [`hll`](../../query/hll-aggfunction.md)
* [`make_set`](../../query/makeset-aggfunction.md)
* [`make_list`](../../query/makelist-aggfunction.md)
* [`percentile`, `percentiles`](../../query/percentiles-aggfunction.md)

## <a name="performance-tips"></a>性能提示

* 当按具体化视图维度之一（聚合依据子句）进行筛选时，具体化视图查询筛选器会得到优化。 如果你知道查询模式经常按某个列（可以是具体化视图中的维度）筛选，请将其包含在视图中。 例如：对于通过 `ResourceId` 公开了 `arg_max` 且经常会按 `SubscriptionId` 进行筛选的具体化视图，建议如下所示：

    **建议做法** ：
    
    ```kusto
    .create materialized-view ArgMaxResourceId on table FactResources
    {
        FactResources | summarize arg_max(Timestamp, *) by SubscriptionId, ResouceId 
    }
    ``` 
    
    **禁止做法** ：
    
    ```kusto
    .create materialized-view ArgMaxResourceId on table FactResources
    {
        FactResources | summarize arg_max(Timestamp, *) by ResouceId 
    }
    ```

* 不要在具体化视图定义中包括转换、规范化和其他可移动到[更新策略](../updatepolicy.md)的繁重计算， 而要在更新策略中执行所有这些过程，仅在具体化视图中执行聚合。 使用此过程在维度表中查找（如果适用）。

    **建议做法** ：
    
    * 更新策略：
    
    ```kusto
    .alter-merge table Target policy update 
    @'[{"IsEnabled": true, 
        "Source": "SourceTable", 
        "Query": 
            "SourceTable 
            | extend ResourceId = strcat('subscriptions/', toupper(SubscriptionId), '/', resourceId)", 
        "IsTransactional": false}]'  
    ```
        
    * 具体化视图：
    
    ```kusto
    .create materialized-view Usage on table Events
    {
    &nbsp;     Target 
    &nbsp;     | summarize count() by ResourceId 
    }
    ```
    
    **禁止做法** ：
    
    ```kusto
    .create materialized-view Usage on table SourceTable
    {
    &nbsp;     SourceTable 
    &nbsp;     | extend ResourceId = strcat('subscriptions/', toupper(SubscriptionId), '/', resourceId)
    &nbsp;     | summarize count() by ResourceId
    }
    ```

> [!NOTE]
> 如果你需要最佳查询时间性能，但可以牺牲一些数据时效性，请使用 [materialized_view() 函数](../../query/materialized-view-function.md)。

## <a name="limitations-on-creating-materialized-views"></a>创建具体化视图的限制

* 无法通过以下方式创建具体化视图：
    * 基于另一个具体化视图。
    * 基于[追随者数据库](../../../follower.md)。 追随者数据库是只读的，而具体化视图需要执行写入操作。  在领导者数据库上定义的具体化视图可以从其追随者进行查询，就像领导者中的任何其他表一样。 
* 具体化视图的源表：
    * 必须是使用[引入方法](../../../ingest-data-overview.md#ingestion-methods-and-tools)之一、使用[更新策略](../updatepolicy.md)或使用[“从查询中引入”命令](../data-ingestion/ingest-from-query.md)直接将内容引入到其中的表。
        * 具体而言，不支持使用[移动区](../move-extents.md)从其他表移动到具体化视图的源表中。 移动区可能会失败并出现以下错误：`Cannot drop/move extents from/to table 'TableName' since Materialized View 'ViewName' is currently processing some of these extents`。 
    * 必须启用 [IngestionTime 策略](../ingestiontimepolicy.md)（默认为启用）。
    * 无法为流式引入启用。
    * 不能是受限制的表或启用了行级安全性的表。
* 不能基于具体化视图使用[游标函数](../databasecursor.md#cursor-functions)。
* 不支持从具体化视图连续导出。

## <a name="cancel-materialized-view-creation"></a>取消具体化视图创建操作

使用 `backfill` 选项时，取消创建具体化视图的过程。 当创建时间太长，你想要在它运行时中止它时，此操作非常有用。  

> [!WARNING]
> 运行此命令后，无法还原具体化视图。

无法立即中止创建进程。 取消命令会通知具体化停止，创建时会定期检查是否请求了取消。 取消命令会等待最长 10 分钟时间，直到具体化视图创建过程被取消，并报告取消是否已成功。 即使取消未在 10 分钟内成功完成，并且取消命令报告了失败，具体化视图也很可能会稍后在创建过程中自行中止。 [.show operations](../operations.md#show-operations) 命令会指示操作是否已取消。 `cancel operation` 命令仅支持取消具体化视图的创建操作，而不支持取消任何其他操作。

### <a name="syntax"></a>语法

`.cancel` `operation` *operationId*

### <a name="properties"></a>属性

|properties|类型|说明
|----------------|-------|---|
|operationId|GUID|从 create materialized-view 命令返回的操作 ID。|

### <a name="output"></a>输出

|输出参数 |类型 |说明
|---|---|---
|OperationId|Guid|create materialized view 命令的操作 ID。
|操作|String|操作类型。
|StartedOn|datetime|创建操作的开始时间。
|CancellationState|string|`Cancelled successfully`（创建被取消）、`Cancellation failed`（等待取消超时）、`Unknown`（视图创建操作不再运行，但未被此操作取消）中的一个。
|ReasonPhrase|string|取消操作失败的原因。

### <a name="example"></a>示例

<!-- csl -->
```
.cancel operation c4b29441-4873-4e36-8310-c631c35c916e
```

|OperationId|操作|StartedOn|CancellationState|ReasonPhrase|
|---|---|---|---|---|
|c4b29441-4873-4e36-8310-c631c35c916e|MaterializedViewCreateOrAlter|2020-05-08 19:45:03.9184142|成功取消||

如果取消操作未在 10 分钟内完成，`CancellationState` 会指示失败。 然后，可能会中止创建操作。
