---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: Azure 事件中心 - Azure Databricks
description: 了解如何使用 Azure 事件中心作为 Azure Databricks 中流式传输数据的源和接收器。
ms.openlocfilehash: 25bdbc3428bb1a65dabe69219a7f6cd1347af74b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473056"
---
# <a name="azure-event-hubs"></a>Azure 事件中心

[Azure 事件中心](https://azure.microsoft.com/services/event-hubs/)是超大规模的遥测引入服务，可收集、转换和存储数百万的事件。 作为分布式流式处理平台，它为用户提供低延迟和可配置的时间保留，使用户可以将大量遥测数据引入到云中，并使用发布-订阅语义从多个应用程序中读取数据。

本文介绍了如何通过 Azure 事件中心和 Azure Databricks 群集使用结构化流式处理。

## <a name="requirements"></a>要求

有关当前的版本支持，请参阅 Azure 事件中心 Spark 连接器项目[自述文件](https://github.com/Azure/azure-event-hubs-spark/blob/master/README.md#latest-releases)中的“最新版本”。

1. 使用 Maven 坐标 `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.17` 在 Azure Databricks 工作区中[创建库](../../../libraries/workspace-libraries.md#maven-libraries)。

   > [!NOTE]
   >
   > 此连接器会定期更新，并且可能会有最新版本：建议你从 [Maven 存储库](https://mvnrepository.com/artifact/com.microsoft.azure/azure-eventhubs-spark)拉取最新的连接器

2. [将创建的库安装到](../../../libraries/cluster-libraries.md#install-libraries)群集中。

## <a name="schema"></a>架构

记录的架构为：

| 列           | 类型               |
|------------------|--------------------|
| body             | binary             |
| partition        | string             |
| offset           | string             |
| sequenceNumber   | long               |
| enqueuedTime     | timestamp          |
| publisher        | string             |
| partitionKey     | string             |
| properties       | map[string,json]   |

`body` 始终以字节数组的形式提供。 请使用 `cast("string")` 显式反序列化 `body` 列。

## <a name="quick-start"></a>快速启动

让我们从一个快速示例开始：WordCount。 以下笔记本演示了如何通过 Azure 事件中心使用结构化流式处理来运行 WordCount。

### <a name="azure-event-hubs-wordcount-with-structured-streaming-notebook"></a>采用结构化流式处理笔记本的 Azure 事件中心 WordCount

[获取笔记本](../../../_static/notebooks/structured-streaming-event-hubs-wordcount.html)

## <a name="configuration"></a>配置

本部分介绍了使用事件中心时所需的配置设置。

有关使用 Azure 事件中心配置结构化流式处理的详细指南，请参阅 Microsoft 制定的[结构化流式处理和 Azure 事件中心集成指南](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md)。

有关使用结构化流式处理的详细指南，请参阅[结构化流式处理](index.md)。

### <a name="connection-string"></a>连接字符串

需要使用事件中心连接字符串来连接到事件中心服务。 可以使用 [Azure 门户](https://portal.azure.com)或使用库中的 `ConnectionStringBuilder` 获取事件中心实例的连接字符串。

#### <a name="azure-portal"></a>Azure 门户

从 Azure 门户获取连接字符串时，它可能有也可能没有 `EntityPath` 键。 请注意以下几点：

```scala
  // Without an entity path
val without = "Endpoint=<endpoint>;SharedAccessKeyName=<key-name>;SharedAccessKey=<key>"

// With an entity path
val with = "Endpoint=sb://<sample>;SharedAccessKeyName=<key-name>;SharedAccessKey=<key>;EntityPath=<eventhub-name>"
```

若要连接到事件中心，必须提供 `EntityPath`。 如果你的连接字符串中没有该路径，请不要担心。
下面的对象将处理此问题：

```scala
import org.apache.spark.eventhubs.ConnectionStringBuilder

val connectionString = ConnectionStringBuilder(without)   // defined in the previous code block
  .setEventHubName("<eventhub-name>")
  .build
```

#### <a name="connectionstringbuilder"></a>ConnectionStringBuilder

另外，也可以使用 `ConnectionStringBuilder` 来创建连接字符串。

```scala
import org.apache.spark.eventhubs.ConnectionStringBuilder

val connectionString = ConnectionStringBuilder()
  .setNamespaceName("<namespace-name>")
  .setEventHubName("<eventhub-name>")
  .setSasKeyName("<key-name>")
  .setSasKey("<key>")
  .build
```

### <a name="eventhubsconf"></a>EventHubsConf

与事件中心相关的所有配置都在 `EventHubsConf` 中进行。 若要创建 `EventHubsConf`，必须传递连接字符串：

```scala
val connectionString = "<event-hub-connection-string>"
val eventHubsConf = EventHubsConf(connectionString)
```

若要详细了解如何获取有效的连接字符串，请参阅[连接字符串](#connection-string)。

有关配置的完整列表，请参阅 [EventHubsConf](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md#eventhubsconf)。 若要开始，请参阅下面的部分配置。

| 选项                  | 值              | 默认                      | 查询类型              | 说明                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------------------|--------------------|------------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| consumerGroup           | 字符串             | “$Default”                   | 流式处理和批处理     | 使用者组是整个事件中心的视图。 使用者组使多个消费应用程序都有各自独立的事件流视图，并按自身步调和偏移量独立读取流。 [Microsoft 文档](/event-hubs/event-hubs-features#event-consumers)中提供了详细信息。 |
| startingPosition        | EventPosition      | 流的开头              | 流式处理和批处理     | 结构化流式处理作业的起始位置。 有关选项读取顺序的信息，请参阅 [startingPositions](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md#eventhubsconf)。                                                                                                                       |
| maxEventsPerTrigger     | long               | partitionCount<br><br>* 1000 | 流式处理查询         | 每个触发器间隔处理的最大事件数的速率限制。 指定的事件总数将按比例分配到不同卷的分区中。                                                                                                                                                                                                                 |

对于每个选项，`EventHubsConf` 中都存在一个相应的设置。 例如： 。

```scala
import org.apache.spark.eventhubs.

val cs = "<your-connection-string>"
val eventHubsConf = EventHubsConf(cs)
  .setConsumerGroup("sample-cg")
  .setMaxEventsPerTrigger(10000)
```

#### <a name="eventposition"></a>EventPosition

`EventHubsConf` 允许用户通过 `EventPosition` 类指定起始（和结束）位置。 `EventPosition` 定义事件在事件中心分区中的位置。 位置可以是排队的时间、偏移量、序列号、流的开头或流的末尾。

```scala
import org.apache.spark.eventhubs._

EventPosition.fromOffset("246812")          // Specifies offset 246812
EventPosition.fromSequenceNumber(100L)      // Specifies sequence number 100
EventPosition.fromEnqueuedTime(Instant.now) // Specifies any event after the current time
EventPosition.fromStartOfStream             // Specifies from start of stream
EventPosition.fromEndOfStream               // Specifies from end of stream
```

如果要在特定位置开始（或结束），只需创建正确的 `EventPosition` 并在 `EventHubsConf` 中对它进行设置即可：

```scala
val connectionString = "<event-hub-connection-string>"
val eventHubsConf = EventHubsConf(connectionString)
  .setStartingPosition(EventPosition.fromEndOfStream)
```

## <a name="production-structured-streaming-with-azure-event-hubs"></a>采用 Azure 事件中心的生产结构化流式处理

与简单地将笔记本附加到群集并以交互方式运行流式处理查询相比，在生产环境中运行流式处理查询时，你可能需要更强的可靠性和正常运行时间保证。 导入并运行以下笔记本，以便演示如何使用 Azure 事件中心和 Azure Databricks 在生产环境中配置和运行结构化流式处理。

有关详细信息，请参阅[生产中的结构化流式处理](production.md)。

### <a name="production-structured-streaming-with-azure-event-hubs-notebook"></a>采用 Azure 事件中心笔记本的生产结构化流式处理

[获取笔记本](../../../_static/notebooks/structured-streaming-event-hubs-integration.html)

## <a name="end-to-end-event-hubs-streaming-tutorial"></a>端到端事件中心流式处理教程

有关使用事件中心将数据流式传输到群集的端到端示例，请参阅[教程：使用事件中心将数据流式传输到 Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/databricks-stream-from-eventhubs)。