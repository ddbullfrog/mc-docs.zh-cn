---
title: 适用于 Azure Functions 的 Azure IoT 中心输出绑定
description: 了解如何使用 Azure Functions 将消息写入 Azure IoT 中心流。
author: craigshoemaker
ms.topic: reference
ms.date: 03/02/2020
ms.author: v-junlch
ms.openlocfilehash: f94f3bf4ff7bc9f45440153b76f9e66f8d0a299f
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211445"
---
# <a name="azure-iot-hub-output-binding-for-azure-functions"></a>适用于 Azure Functions 的 Azure IoT 中心输出绑定

本文介绍如何使用 IoT 中心的 Azure Functions 输出绑定。 IoT 中心支持基于 [Azure 事件中心绑定](functions-bindings-event-hubs.md)。

若要了解设置和配置详细信息，请参阅[概述](functions-bindings-event-iot.md)。

> [!IMPORTANT]
> 虽然下述代码示例使用事件中心 API，但给定的语法适用于 IoT 中心函数。

[!INCLUDE [functions-bindings-event-hubs](../../includes/functions-bindings-event-hubs-output.md)]

## <a name="next-steps"></a>后续步骤

- [响应发送到事件中心事件流的事件（触发器）](./functions-bindings-event-iot-trigger.md)

