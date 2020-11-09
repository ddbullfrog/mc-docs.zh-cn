---
title: 盘区合并策略 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的盘区合并策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1c40c037dacea3d3bdc82fb3324923e5ec7b54cd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106117"
---
# <a name="merge-policy"></a>合并策略

合并策略定义是否应合并以及应如何合并 Kusto 群集中的[盘区（数据分片）](../management/extents-overview.md)。

合并操作有两种类型：`Merge`，此操作会重新生成索引；以及 `Rebuild`，此操作会完全重新引入数据。

这两种操作类型都会生成单个盘区来替换源盘区。

默认情况下，首选重新生成操作。 如果某些盘区不符合进行重新生成的条件，则将尝试合并它们。  

> [!NOTE]
> * 即使已设置合并策略，使用不同的 `drop-by` 标记来标记盘区也将导致此类盘区无法进行合并。 有关详细信息，请参阅[盘区标记](../management/extents-overview.md#extent-tagging)。
> * 不会合并标记并集超过一百万个字符长度的盘区。
> * 数据库或表的[分片策略](./shardingpolicy.md)也会对盘区合并的方式产生一定的影响。

## <a name="merge-policy-properties"></a>合并策略属性

合并策略包含以下属性：

* **RowCountUpperBoundForMerge** ：
    * 默认值：
      * 对于 2020 年 6 月以前设置的策略，默认值为 0（无限制）。
      * 对于自 2020 年 6 月起设置的策略，默认值为 16,000,000。
    * 合并盘区允许的最大行数。
    * 适用于合并操作，不适用于重新生成。  
* **OriginalSizeMBUpperBoundForMerge** ：
    * 默认为 0（无限制）。
    * 合并盘区允许的最大原始大小 (MB)。
    * 适用于合并操作，不适用于重新生成。  
* **MaxExtentsToMerge** ：
    * 默认为 100。
    * 允许在单个操作中合并的最大盘区数量。
    * 适用于合并操作。
* **LoopPeriod** ：
    * 默认为 01:00:00（1 小时）。
    * 数据管理服务在开始合并或重新生成操作的两次连续迭代之间等待的最长时间。
    * 适用于合并和重新生成操作。
* **AllowRebuild** ：
    * 默认为“true”。
    * 定义是否已启用 `Rebuild` 操作（在这种情况下，其优先级高于 `Merge` 操作）。
* **AllowMerge** ：
    * 默认为“true”。
    * 定义是否已启用 `Merge` 操作，在这种情况下，其优先级低于 `Rebuild` 操作。
* **MaxRangeInHours** ：
    * 默认为 8。
        * [具体化视图](materialized-views/materialized-view-overview.md)中默认保留 14 天，除非在具体化视图的有效[保留策略](retentionpolicy.md)中禁用了可恢复性。
    * 最大允许的任意两个不同盘区的创建时间之间的差异（以小时为单位），如超过，将无法进行合并。
    * 时间戳源于盘区创建，且不与盘区中包含的实际数据相关。
    * 适用于合并和重新生成操作。
    * 应根据有效的[保留策略](./retentionpolicy.md)“SoftDeletePeriod”或[缓存策略](./cachepolicy.md)“DataHotSpan”值来设置此值 。 取 SoftDeletePeriod 和 DataHotSpan 中的较低值 。 将 MaxRangeInHours 值设置为在其 2-3% 之间。 请参阅[示例](#maxrangeinhours-examples)。

## <a name="maxrangeinhours-examples"></a>MaxRangeInHours 示例

|最小 [SoftDeletePeriod（保留策略），DataHotSpan（缓存策略）]|最大范围（合并策略）（以小时为单位）|
|--------------------------------------------------------------------|---------------------------------|
|7 天（168 小时）                                                  | 4                               |
|14 天（336 小时）                                                 | 8                               |
|30 天（720 小时）                                                 | 18                              |
|60 天（1,440 小时）                                               | 36                              |
|90 天（2,160 小时）                                               | 60                              |
|180 天（4,320 小时）                                              | 120                             |
|365 天（8,760 小时）                                              | 250                             |

> [!WARNING]
> 更改盘区合并策略前，请咨询 Azure 数据资源管理器团队。

创建数据库时，将使用上面提到的默认合并策略值设置该数据库。 默认情况下，在数据库中创建的所有表都将继承该策略，除非其策略在表级别被显式替代。

有关详细信息，请参阅[可用于管理数据库或表合并策略的控制命令](../management/merge-policy.md)。
