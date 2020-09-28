---
title: Azure 媒体服务诊断日志架构 - Azure
description: 本文演示 Azure 媒体服务诊断日志架构。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: 2cf2ed1b0dee9fa7085e4b3efa0f7165cb15b911
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245638"
---
# <a name="diagnostic-logs-schemas"></a>诊断日志架构

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Google Widevine 内容保护服务目前在 Azure 中国区域不可用。

可以通过 [Azure Monitor](../../azure-monitor/overview.md) 监视指标和诊断日志，以便了解应用程序的执行情况。 可以监视媒体服务诊断日志，并针对收集的指标和日志创建警报与通知。 可将日志发送到 [Azure 存储](/storage/)、将其流式传输到 [Azure 事件中心](/event-hubs/)、将其导出到 [Log Analytics](/azure-monitor/)，或使用第三方服务。

有关详细信息，请参阅 [Azure Monitor 指标](../../azure-monitor/platform/data-platform.md)和 [Azure Monitor 诊断日志](../../azure-monitor/platform/platform-logs-overview.md)。

本文介绍媒体服务诊断日志架构。

## <a name="top-level-diagnostic-logs-schema"></a>顶级诊断日志架构

有关顶级诊断日志架构的详细说明，请参阅 [Azure 诊断日志支持的服务、架构和类别](../../azure-monitor/platform/resource-logs-schema.md)。

## <a name="key-delivery-log-schema"></a>密钥传送日志架构

### <a name="properties"></a>属性

这些属性特定于密钥传送日志架构。

|名称|说明|
|---|---|
|keyId|请求的密钥的 ID。|
|keyType|可能是以下值之一：“Clear”（不加密）、“FairPlay”或“PlayReady”。|
|policyName|策略的 Azure 资源管理器名称。|
|tokenType|令牌类型。|
|statusMessage|状态消息。|

### <a name="examples"></a>示例

密钥传送请求架构的属性。

```json
{
    "time": "2019-01-11T17:59:10.4908614Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Succeeded",
    "resultSignature": "OK",
    "durationMs": 315,
    "identity": {
        "authorization": {
            "issuer": "http://testacs",
            "audience": "urn:test"
        },
        "claims": {
            "urn:microsoft:azure:mediaservices:contentkeyidentifier": "3321e646-78d0-4896-84ec-c7b98eddfca5",
            "iss": "http://testacs",
            "aud": "urn:test",
            "exp": "1547233138"
        }
    },
    "level": "Informational",
    "location": "chinaeast2",
    "properties": {
        "requestId": "b0243468-d8e5-4edf-a48b-d408e1661050",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "JWT",
        "statusMessage": "OK"
    }
} 
```

```json
 {
    "time": "2019-01-11T17:59:33.4676382Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Failed",
    "resultSignature": "Unauthorized",
    "durationMs": 2,
    "level": "Error",
    "location": "chinaeast2",
    "properties": {
        "requestId": "875af030-b77c-416b-b7e1-58f23ebec182",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "None",
        "statusMessage": "No token present in authorization header or URL."
    }
} 
```

## <a name="next-steps"></a>后续步骤

[监视媒体服务指标和诊断日志](media-services-metrics-diagnostic-logs.md)
