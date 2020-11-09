---
title: 数据分区策略 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的数据分区策略，以及如何使用它来提高查询性能。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 06/10/2020
ms.date: 10/29/2020
ms.openlocfilehash: 46385a53f3c35548e2fffe760c14e0b4068366be
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104218"
---
# <a name="partitioning-policy"></a>分区策略

分区策略定义了对于特定表，是否以及如何对[区（数据分片）](../management/extents-overview.md)进行分区。

该策略的主要目的是提高可以缩小数据集的查询的性能，例如，基于分区列进行筛选的查询、使用聚合的查询，或基于高基数字符串列进行联接的查询。 该策略还可能会导致更好的数据压缩。

> [!CAUTION]
> 对于定义了分区策略的表数量，没有设置硬编码限制。 但是，每个附加表都会增加在群集节点上运行的后台数据分区过程的开销。 添加表可能会导致使用更多的群集资源。 有关详细信息，请参阅[监视](#monitor-partitioning)和[容量](#partition-capacity)。

## <a name="partition-keys"></a>分区键

支持以下类型的分区键。

|种类                                                   |列类型 |分区属性                    |分区值                                        |
|-------------------------------------------------------|------------|----------------------------------------|----------------------|
|[哈希](#hash-partition-key)                            |`string`    |`Function`, `MaxPartitionCount`, `Seed` | `Function`(`ColumnName`, `MaxPartitionCount`, `Seed`) |
|[统一范围](#uniform-range-datetime-partition-key) |`datetime`  |`RangeSize`, `Reference`                | `bin_at`(`ColumnName`, `RangeSize`, `Reference`)      |

### <a name="hash-partition-key"></a>哈希分区键

> [!NOTE]
> 仅在以下实例中对表中的 `string` 类型列应用哈希分区键：
> * 大多数查询都使用相等筛选器 (`==`, `in()`)。
> * 大多数查询在大维度（基数为 10M 或更高）的特定 `string` 类型列（例如 `application_ID`、`tenant_ID` 或 `user_ID`）上聚合/联接。

* 哈希取模函数用于对数据进行分区。
* 同类（已分区）盘区中的数据按哈希分区键排序。
  * 如果表上定义了哈希分区键，则不需要在[行顺序策略](roworderpolicy.md)中包含哈希分区键。
* 如果查询使用[无序策略](../query/shufflequery.md)，并且按照此策略，`join`、`summarize` 或 `make-series` 中使用的 `shuffle key` 是表的哈希分区键，那么查询的性能将会更好，因为需要跨群集节点移动的数据量减少。

#### <a name="partition-properties"></a>分区属性

|properties | 说明 | 支持的值| 建议的值 |
|---|---|---|---|
| `Function` | 要使用的哈希取模函数的名称。| `XxHash64` | |
| `MaxPartitionCount` | 每个时间段要创建的最大分区数（哈希取模函数的取模参数）。 | 在 `(1,2048]` 范围内。 <br>  大于群集中节点数的 5 倍，小于列的基数。 |  值越大，群集节点上数据分区过程的开销就越大，每个时间段的区数量也就越大。 对于少于 50 个节点的群集，请从 `256` 开始。 基于这些注意事项，或者基于查询性能的优势与数据引入后分区的开销，调整该值。
| `Seed` | 用于将哈希值随机化。 | 正整数。 | `1`，这也是默认值。 |
| `PartitionAssignmentMode` | 用于将分区分配给群集中节点的模式。 | `Default`：属于同一分区的所有同类（已分区）盘区都分配给同一个节点。 <br> `Uniform`：将忽略区的分区值。 区将统一分配给群集的节点。 | 如果查询不根据哈希分区键联接或聚合，请使用 `Uniform`。 否则使用 `Default`。 |

#### <a name="hash-partition-key-example"></a>哈希分区键示例

名为 `tenant_id` 的 `string` 类型列的哈希分区键。
它使用 `XxHash64` 哈希函数，其中 `MaxPartitionCount` 为 `256`，默认的 `Seed` 为 `1`。

```json
{
  "ColumnName": "tenant_id",
  "Kind": "Hash",
  "Properties": {
    "Function": "XxHash64",
    "MaxPartitionCount": 256,
    "Seed": 1,
    "PartitionAssignmentMode": "Default"
  }
}
```

### <a name="uniform-range-datetime-partition-key"></a>统一范围日期/时间分区键

> [!NOTE] 
> 只有当引入到表中的数据不太可能根据此列排序时，才对表中的 `datetime` 类型的列应用统一范围的日期/时间分区键。

在这些情况下，你可以在区之间重新组织数据，以便每个区都包括有限时间范围的记录。 此过程会使 `datetime` 列上的筛选器在查询时更有效。

使用的分区函数是 [bin_at()](../query/binatfunction.md)，该函数不可自定义。

#### <a name="partition-properties"></a>分区属性

|properties | 说明 | 建议的值 |
|---|---|---|---|
| `RangeSize` | 一个 `timespan` 标量常量，指示每个日期/时间分区的大小。 | 从值 `1.00:00:00`（一天）开始。 不要设置更短的值，因为这可能会导致表中有大量无法合并的小区。
| `Reference` | 一个 `datetime` 标量常量，指示一个固定的时间点，根据该时间点调整日期/时间分区。 | 从 `1970-01-01 00:00:00` 开始。 如果存在日期/时间分区键具有 `null` 值的记录，则将其分区值设置为 `Reference`。 |

#### <a name="uniform-range-datetime-partition-example"></a>统一范围日期/时间分区示例

代码片段在名为 `timestamp` 的 `datetime` 类型列上显示了统一日期/时间范围分区键。
它使用 `datetime(1970-01-01)` 作为参考点，每个分区的大小为 `1d`。

```json
{
  "ColumnName": "timestamp",
  "Kind": "UniformRange",
  "Properties": {
    "Reference": "1970-01-01T00:00:00",
    "RangeSize": "1.00:00:00"
  }
}
```

## <a name="the-policy-object"></a>策略对象

表的数据分区策略默认为 `null`，在这种情况下，将不会对表中的数据进行分区。

数据分区策略具有以下主要属性：

* **PartitionKeys** ：
  * [分区键](#partition-keys)的集合，用于定义如何对表中的数据进行分区。
  * 一个表最多可以有 `2` 个分区键，并有下列选项之一：
    * 一个[哈希分区键](#hash-partition-key)。
    * 一个[统一范围日期/时间分区键](#uniform-range-datetime-partition-key)。
    * 一个[哈希分区键](#hash-partition-key)和一个[统一范围日期/时间分区键](#uniform-range-datetime-partition-key)。
  * 每个分区键都具有以下属性：
    * `ColumnName`：`string ` - 将根据其对数据进行分区的列的名称。
    * `Kind`：`string` - 要应用的数据分区类型（`Hash` 或 `UniformRange`）。
    * `Properties`：`property bag` -根据完成的分区定义参数。

* **EffectiveDateTime** ：
  * 策略生效的 UTC 日期/时间。
  * 此属性是可选的。 如果未指定它，则策略将对应用策略后引入的数据生效。
  * 分区过程会忽略由于保留而可能丢弃的任何非同类（非分区）区。 这些区会被忽略，因为它们的创建时间早于表的有效软删除周期的 90%。
    > [!NOTE]
    > 你可以设置过去的一个日期/时间值，并对已引入的数据分区。 但是，这种做法可能会显著增加分区过程中使用的资源。 

### <a name="data-partitioning-example"></a>数据分区示例

具有两个分区键的数据分区策略对象。
1. 名为 `tenant_id` 的 `string` 类型列的哈希分区键。
    * 它使用 `XxHash64` 哈希函数，其中 `MaxPartitionCount` 为 256，默认的 `Seed` 为 `1`。
1. 名为 `timestamp` 的 `datetime` 类型列的统一日期/时间范围分区键。
    * 它使用 `datetime(1970-01-01)` 作为参考点，每个分区的大小为 `1d`。

```json
{
  "PartitionKeys": [
    {
      "ColumnName": "tenant_id",
      "Kind": "Hash",
      "Properties": {
        "Function": "XxHash64",
        "MaxPartitionCount": 256,
        "Seed": 1,
        "PartitionAssignmentMode": "Default"
      }
    },
    {
      "ColumnName": "timestamp",
      "Kind": "UniformRange",
      "Properties": {
        "Reference": "1970-01-01T00:00:00",
        "RangeSize": "1.00:00:00"
      }
    }
  ]
}
```

## <a name="additional-properties"></a>其他属性

可以将以下属性定义为策略的一部分。 这些属性是可选的，建议不要更改它们。

|properties | 说明 | 建议的值 | 默认值 |
|---|---|---|---|
| **MinRowCountPerOperation** |  单个数据分区操作的源盘区行数总和的最小目标值。 | | `0` |
| **MaxRowCountPerOperation** |  单个数据分区操作的源盘区行数总和的最大目标值。 | 如果看到单个分区操作消耗了大量的内存或 CPU，请设置一个小于 5M 的值。 有关详细信息，请参阅[监视](#monitor-partitioning)。 | `0`，默认目标是 5,000,000 条记录。 |

## <a name="the-data-partitioning-process"></a>数据分区过程

* 数据分区在群集中作为引入后的后台进程运行。
  * 持续引入的表始终具有尚未分区的数据的“尾部”（非同类盘区）。
* 无论策略中 `EffectiveDateTime` 属性的值如何，数据分区只在热盘区上运行。
  * 如果需要对冷区进行分区，则需要临时调整[缓存策略](cachepolicy.md)。

## <a name="monitor-partitioning"></a>监视分区

使用 [.show diagnostics](../management/diagnostics.md#show-diagnostics) 命令监视群集中分区的进度或状态。

```kusto
.show diagnostics
| project MinPartitioningPercentageInSingleTable, TableWithMinPartitioningPercentage
```

输出包括：

  * `MinPartitioningPercentageInSingleTable`：群集中具有数据分区策略的所有表中分区数据的最小百分比。
    * 如果此百分比持续低于 90%，则评估群集的分区[容量](partitioningpolicy.md#partition-capacity)。
  * `TableWithMinPartitioningPercentage`：上面显示了分区百分比的表的完全限定名。

使用 [.show 命令](commands.md)监视分区命令及其资源使用情况。 例如：

```kusto
.show commands 
| where StartedOn > ago(1d)
| where CommandType == "ExtentsPartition"
| parse Text with ".partition async table " TableName " extents" *
| summarize count(), sum(TotalCpu), avg(tolong(ResourcesUtilization.MemoryPeak)) by TableName, bin(StartedOn, 15m)
| render timechart with(ysplit = panels)
```

## <a name="partition-capacity"></a>分区容量

* 数据分区过程会导致创建更多的盘区。 群集可以逐渐增加其[区合并容量](../management/capacitypolicy.md#extents-merge-capacity)，以便[合并区](../management/extents-overview.md)的过程可以赶上进度。
* 如果有较高的引入吞吐量，或者有足够多的表定义了分区策略，则群集可能会逐渐增加其[盘区分区容量](../management/capacitypolicy.md#extents-partition-capacity)，以便[分区盘区的过程](#the-data-partitioning-process)可以保持。
* 为了避免消耗过多资源，对这些动态增加进行了限制。 如果它们全部耗尽，则可能需要逐渐线性地增加，以超过上限。
  * 如果增加容量会显著增加群集资源的使用，则可以手动或通过启用自动缩放来[横向](../../manage-cluster-vertical-scaling.md)/[纵向扩展群集](../../manage-cluster-horizontal-scaling.md)。

## <a name="outliers-in-partitioned-columns"></a>已分区列中的离群值

* 以下情况可能会导致群集节点上的数据分布不均衡，并降低查询性能：
    * 如果某个哈希分区键包含比其他值要普遍得多的值，例如，空字符串或泛型值（例如 `null` 或 `N/A`）。
    * 这些值表示在数据集中更为普遍的实体（例如 `tenant_id`）。
* 如果统一范围日期/时间分区键包含的大部分值与列中的大多数值“相差甚远”，则会增加数据分区过程的开销，并可能会导致群集需要跟踪多个小区。 这种情况的一个示例是来自遥远的过去或未来的日期/时间值。

在这两种情况下，要么“修复”数据，要么在数据引入之前或引入时筛选掉数据中不相关的记录，以减少群集上数据分区的开销。 例如，使用[更新策略](updatepolicy.md)。

## <a name="next-steps"></a>后续步骤

使用[分区策略控制命令](../management/partitioning-policy.md)管理表的数据分区策略。
