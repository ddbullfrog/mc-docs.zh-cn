---
title: include 文件
description: include 文件
services: digital-twins
ms.service: digital-twins
ms.topic: include
ms.date: 08/05/2020
author: deepakpalled
ms.author: v-junlch
manager: diviso
ms.custom: include file
ms.openlocfilehash: 729e5ada72a786ef05e219676ffbe88732861a06
ms.sourcegitcommit: 36e7f37481969f92138bfe70192b1f4a2414caf7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/05/2020
ms.locfileid: "91427735"
---
### <a name="property-limits"></a>属性限制

在第 1 代中，Azure 时序见解属性限制已从最大上限 800 增加到 1,000。 提供的事件属性具有相应的 JSON、CSV 和图表列。

| SKU | 最大属性数 |
| --- | --- |
| 第 2 代 (L1) | 1,000 属性（列） |
| 第 1 代 (S1) | 600 属性（列） |
| 第 1 代 (S2) | 800 属性（列） |

### <a name="event-sources"></a>事件源

每个实例最多支持两个事件源。

* 了解如何[添加事件中心源](/time-series-insights/how-to-ingest-data-event-hub)。
* 配置 [IoT 中心源](/time-series-insights/how-to-ingest-data-iot-hub)。

默认情况下，[第 2 代环境支持的入口速率](/time-series-insights/concepts-streaming-ingress-throughput-limits)最大为每环境每秒 1 兆字节（MB/秒）。 如果需要，客户可以将其环境扩展到 16 MB/秒的吞吐量。 还存在每个分区 0.5 MB/秒的限制。

### <a name="api-limits"></a>API 限制

[REST API 参考文档](https://docs.microsoft.com/rest/api/time-series-insights/preview#limits-1)中指定了针对 Azure 时序见解第 2 代的 REST API 限制。

