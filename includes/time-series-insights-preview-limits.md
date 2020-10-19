---
title: include 文件
description: include 文件
services: digital-twins
ms.service: digital-twins
ms.topic: include
ms.date: 09/28/2020
author: deepakpalled
ms.author: v-junlch
manager: diviso
ms.custom: include file
ms.openlocfilehash: 611900c5cd40d0401a0ae6ef05982a7decdc9267
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937409"
---
### <a name="property-limits"></a>属性限制

在第 1 代中，Azure 时序见解属性限制已从最大上限 800 增加到 1,000。 提供的事件属性具有相应的 JSON、CSV 和图表列，可以在 Azure 时序见解第 2 代资源管理器中查看它们。

| SKU | 最大属性数 |
| --- | --- |
| 第 2 代 (L1) | 1,000 属性（列） |
| 第 1 代 (S1) | 600 属性（列） |
| 第 1 代 (S2) | 800 属性（列） |

### <a name="streaming-ingestion"></a>流式引入

* 每个环境最多只能有两个[事件源](../articles/time-series-insights/concepts-streaming-ingestion-event-sources.md)。

* 有关事件源的最佳做法和一般指南，可在[此处](../articles/time-series-insights/concepts-streaming-ingestion-event-sources.md#streaming-ingestion-best-practices)找到

* 默认情况下，对于每个 Azure 时序见解第 2 代环境，Azure 时序见解第 2 代可按**每秒最多 1 兆字节 (MBps)** 的速率引入传入的数据。 存在针对[单个中心分区](../articles/time-series-insights/concepts-streaming-ingress-throughput-limits.md#hub-partitions-and-per-partition-limits)的其他限制。 通过 Azure 门户提交支持票证最多可获得 8 MBps 的速率。 若要了解详细信息，请参阅[流式引入吞吐量限制](../articles/time-series-insights/concepts-streaming-ingress-throughput-limits.md)。

### <a name="api-limits"></a>API 限制

[REST API 参考文档](https://docs.microsoft.com/rest/api/time-series-insights/preview#limits-1)中指定了针对 Azure 时序见解第 2 代的 REST API 限制。

