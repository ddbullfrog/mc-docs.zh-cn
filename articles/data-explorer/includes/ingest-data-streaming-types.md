---
title: include 文件
description: include 文件
author: orspod
ms.service: data-explorer
ms.topic: include
origin.date: 07/13/2020
ms.date: 08/18/2020
ms.author: v-tawe
ms.reviewer: alexefro
ms.custom: include file
ms.openlocfilehash: 67433265d5f71975a594b86920332b1e4599bb4b
ms.sourcegitcommit: f4bd97855236f11020f968cfd5fbb0a4e84f9576
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/18/2020
ms.locfileid: "91146369"
---
## <a name="use-streaming-ingestion-to-ingest-data-to-your-cluster"></a>使用流式引入将数据引入到群集

支持两种流式引入类型：

* [事件中心](../ingest-data-event-hub.md)或 [IoT 中心](../ingest-data-iot-hub.md)（用作数据源）。
* “自定义引入”需要编写使用某个 Azure 数据资源管理器[客户端库](../kusto/api/client-libraries.md)之一的应用程序。 有关示例应用程序，请参阅[流式引入示例](https://github.com/Azure/azure-kusto-samples-dotnet/tree/master/client/StreamingIngestionSample)。

### <a name="choose-the-appropriate-streaming-ingestion-type"></a>选择适当的流式引入类型

|条件|事件中心|自定义引入|
|---------|---------|---------|
|启动引入之后、数据可供查询之前的数据延迟 | 延迟更长 | 延迟更短  |
|开发开销 | 快速轻松的设置，不产生开发开销 | 应用程序需要处理错误并确保数据一致性，因此开发开销较高 |
