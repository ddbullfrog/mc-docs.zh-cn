---
title: 从 IoT 中心引入 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的“从 IoT 中心引入”功能。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: how-to
origin.date: 08/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 577e8eb99b74854b9861534c098be96b0d967c35
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104057"
---
# <a name="iot-hub-data-connection"></a>IoT 中心数据连接

[Azure IoT 中心](/iot-hub/about-iot-hub)是一项托管服务，承载在云中，充当中央消息中心，用于 IoT 应用程序与其管理的设备之间的双向通信。 Azure 数据资源管理器使用其[与事件中心兼容的内置终结点](/iot-hub/iot-hub-devguide-messages-d2c#routing-endpoints)提供从客户托管 IoT 中心进行的连续引入。

IoT 引入管道需要完成几个步骤。 首先，创建一个 IoT 中心，并将设备注册到此 IoT 中心。 然后，创建 Azure 数据资源管理器目标表，使用给定的[引入属性](#ingestion-properties)将[特定格式的数据](#data-format)引入到该表中。 IoT 中心连接需要知道[事件路由](#events-routing)才能连接到 Azure 数据资源管理器表。 根据[事件系统属性映射](#event-system-properties-mapping)，使用选定的属性嵌入数据。 可以通过 [Azure 门户](ingest-data-iot-hub.md)使用 [C#](data-connection-iot-hub-csharp.md) 或 [Python](data-connection-iot-hub-python.md) 以编程方式管理此过程，也可以使用 [Azure 资源管理器模板](data-connection-iot-hub-resource-manager.md)来这样做。

有关 Azure 数据资源管理器中数据引入的常规信息，请参阅 [Azure 数据资源管理器数据引入概述](ingest-data-overview.md)。

## <a name="data-format"></a>数据格式

* 将以 [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata?view=azure-dotnet) 对象的形式从事件中心终结点读取数据。
* 请参阅[支持的格式](ingestion-supported-formats.md)。
    > [!NOTE]
    > IoT 中心不支持 .raw 格式。
* 请参阅[支持的压缩](ingestion-supported-formats.md#supported-data-compression-formats)。
  * 原始的未压缩数据大小应该是 blob 元数据的一部分，否则 Azure 数据资源管理器会对其进行估算。 每个文件的引入未压缩大小限制为 4 GB。

## <a name="ingestion-properties"></a>引入属性

引入属性指示引入过程将数据路由到何处以及如何对其进行处理。 可以使用 [EventData.Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties) 指定事件的[引入属性](ingestion-properties.md)。 可以设置以下属性：

|属性 |说明|
|---|---|
| 表 | 现有目标表的名称（区分大小写）。 替代“`Data Connection`”窗格上设置的“`Table`”。 |
| 格式 | 数据格式。 替代“`Data Connection`”窗格上设置的“`Data format`”。 |
| IngestionMappingReference | 要使用的现有[引入映射](kusto/management/create-ingestion-mapping-command.md)的名称。 替代“`Data Connection`”窗格上设置的“`Column mapping`”。|
| 编码 |  数据编码，默认值为 UTF8。 可以是 [.NET 支持的任何编码](https://docs.microsoft.com/dotnet/api/system.text.encoding?view=netframework-4.8#remarks)。 |

> [!NOTE]
> 只有创建数据连接后进入队列的事件才会被引入。

## <a name="events-routing"></a>事件路由

设置到 Azure 数据资源管理器群集的 IoT 中心连接时，请指定目标表属性（表名、数据格式和映射）。 此设置是用于你的数据的默认路由，也称为“静态路由”。
还可以使用事件属性指定每个事件的目标表属性。 连接将按照 [EventData.Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties) 中指定的要求动态路由数据，替代此事件的静态属性。

> [!Note]
> 如果选择了“我的数据包括路由信息”，则必须提供必要的路由信息作为事件属性的一部分。

## <a name="event-system-properties-mapping"></a>事件系统属性映射

系统属性是一个集合，用于存储收到事件时由 IoT 中心服务设置的属性。 Azure 数据资源管理器 IoT 中心连接会将所选属性嵌入置于表中的数据中。

> [!Note]
> 对于 `csv` 映射，属性将按下表中列出的顺序添加到记录的开头。 对于 `json` 映射，将根据下表中的属性名称添加属性。

### <a name="system-properties"></a>系统属性

IoT 中心公开了以下系统属性：

|属性 |说明|
|---|---|
| message-id | 用户可设置的消息标识符，用于请求-答复模式。 |
| sequence-number | IoT 中心分配给每条云到设备消息的编号（对每个设备队列是唯一的）。 |
| to | 在云到设备消息中指定的目标。 |
| absolute-expiry-time | 消息过期的日期和时间。 |
| iothub-enqueuedtime | IoT 中心收到设备到云消息的日期和时间。 |
| correlation-id| 响应消息中的字符串属性，通常包含采用“请求-答复”模式的请求的 MessageId。 |
| user-id| 用于指定消息的源的 ID。 |
| iothub-ack| 反馈消息生成器。 |
| iothub-connection-device-id| IoT 中心对设备到云的消息设置的 ID。 它包含发送了消息的设备的 deviceId。 |
| iothub-connection-auth-generation-id| IoT 中心对设备到云的消息设置的 ID。 它包含发送了消息的设备的 connectionDeviceGenerationId（取决于设备标识属性）。 |
| iothub-connection-auth-method| 由 IoT 中心对设备到云的消息设置的身份验证方法。 此属性包含用于验证发送消息的设备的身份验证方法的相关信息。 |

如果在表的“数据源”部分中选择了“事件系统属性”，则必须在表架构和映射中包含这些属性。

[!INCLUDE [data-explorer-container-system-properties](includes/data-explorer-container-system-properties.md)]

## <a name="iot-hub-connection"></a>IoT 中心连接

> [!Note]
> 为了获得最佳性能，请在 Azure 数据资源管理器群集所在的区域中创建所有资源。

### <a name="create-an-iot-hub"></a>创建 IoT 中心

[创建 IoT 中心](ingest-data-iot-hub.md#create-an-iot-hub)（如果还没有 IoT 中心）。 可以通过 [Azure 门户](ingest-data-iot-hub.md)使用 [C#](data-connection-iot-hub-csharp.md) 或 [Python](data-connection-iot-hub-python.md) 以编程方式管理到 IoT 中心的连接，也可以使用 [Azure 资源管理器模板](data-connection-iot-hub-resource-manager.md)来这样做。

> [!Note]
> * `device-to-cloud partitions` 计数不可更改，因此在设置分区计数时应考虑长期规模。
> * 使用者组对于每个使用者来说必须独一无二。 创建专用于 Azure 数据资源管理器连接的使用者组。 在 Azure 门户中找到你的资源，然后转到“`Built-in endpoints`”，以便添加新的使用者组。

## <a name="sending-events"></a>发送事件

请参阅用于模拟设备并生成数据的[示例项目](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Quickstarts/simulated-device)。

## <a name="next-steps"></a>后续步骤

可以通过多种方法将数据引入 IoT 中心。 有关每种方法的演练，请参阅以下链接。

* [将数据从 IoT 中心引入到 Azure 数据资源管理器](ingest-data-iot-hub.md)
* [使用 C#（预览版）为 Azure 数据资源管理器创建 IoT 中心数据连接](data-connection-iot-hub-csharp.md)
* [使用 Python 为 Azure 数据资源管理器创建 IoT 中心数据连接（预览版）](data-connection-iot-hub-python.md)
* [使用 Azure 资源管理器模板为 Azure 数据资源管理器创建 IoT 中心数据连接](data-connection-iot-hub-resource-manager.md)
