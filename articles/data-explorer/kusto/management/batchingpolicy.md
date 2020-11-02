---
title: Kusto IngestionBatching 策略可优化批处理 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 IngestionBatching 策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: 410a22cb005183448c5c1bbcd1e889398dc4617b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105227"
---
# <a name="ingestionbatching-policy"></a>IngestionBatching 策略

## <a name="overview"></a>概述

在引入过程中，Kusto 试图通过在等待引入时对小的引入数据块执行批处理来优化吞吐量。
这种批处理减少了引入过程消耗的资源，并且不需要资源在引入后优化非批处理引入产生的小型数据分片。

但是，在引入之前执行批处理有一个缺点，就是这么做会产生强制性延迟，这样从请求引入数据到准备好进行查询之间的端到端时间会更长。

若想能够进行权衡，可以使用 `IngestionBatching` 策略。
此策略仅应用于队列中的引入，并在批量处理小型 blob 时提供允许的最大强制延迟。

## <a name="details"></a>详细信息

如上所述，要批量引入的数据有一个最优大小。
当前其大小大约是 1GB 的未压缩数据。 在包含远小于最优大小的数据的 blob 中完成的引入并非最优的，因此在排了队的引入中，Kusto 将把这些小型的 blob 一起进行批处理。 批处理的实施会持续到第一个条件变为 true 为止：

1. 批处理数据的总大小达到最优大小，或
2. 已达到 `IngestionBatching` 策略允许的最大延迟时间、总大小或 blob 数

可以在数据库或表上设置 `IngestionBatching` 策略。 默认情况下，如果未定义策略，Kusto 会为批处理使用以下默认值：最大延迟时间为“5 分钟”，总共“1000”个项，总大小为“1G”  。

> [!WARNING]
> 将此策略设置为非常小的值会增加群集的 COGS 并降低性能。 此外，在限制内，由于并行管理多个引入进程而产生开销，减小该值实际上可能导致增加有效的端到端引入延迟。

## <a name="additional-resources"></a>其他资源

* [IngestionBatching 策略命令参考](../management/batching-policy.md)
* [引入最佳做法 - 针对吞吐量进行优化](../api/netfx/kusto-ingest-best-practices.md#optimizing-for-throughput)