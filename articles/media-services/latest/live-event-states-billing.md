---
title: Azure 媒体服务中的 LiveEvent 状态和计费 | Microsoft Docs
description: 本主题概述了 Azure 媒体服务 LiveEvent 状态和计费。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: fe7fb8dcee5f7a0e021c33947b48753264cb2662
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245120"
---
# <a name="live-event-states-and-billing"></a>直播活动状态和计费

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

在 Azure 媒体服务中，一旦直播活动的状态转换为**正在运行**，就会开始计费。 即使没有视频流经该服务，也会向你收费。 若要停止对直播活动的计费，必须停止直播活动。

将[直播活动](https://docs.microsoft.com/rest/api/media/liveevents)上的 **LiveEventEncodingType** 设置为 Standard 或 Premium1080p 后，媒体服务就会自动关闭在输入源丢失 12 小时后仍处于**正在运行**状态但却没有**实时输出**运行的直播活动。 但是，在直播活动处于**正在运行**状态的时间段内，仍会进行计费。

> [!NOTE]
> 直通直播活动不会自动关闭，必须通过 API 显式停止，以避免过多计费。 

## <a name="states"></a>状态

直播活动可能会处于以下任一状态。

|状态|说明|
|---|---|
|**已停止**| 这是直播活动在创建后的初始状态（除非设置了自动启动）。此状态下不会发生计费。 在此状态下，可以更新直播活动属性，但不允许进行流式处理。|
|**正在启动**| 正在启动直播活动并分配资源。 此状态下不会发生计费。 此状态下不允许进行更新或流式处理。 如果发生错误，则直播活动会返回到“已停止”状态。|
|**正在运行**| 已分配了直播活动资源，已生成了引入和预览 URL，并且能够接收实时流。 此时，计费处于活动状态。 必须显式对直播活动资源调用停止操作才能停止进一步计费。|
|**正在停止**| 正在停止直播活动并解除预配资源。 此暂时性状态下不会发生计费。 此状态下不允许进行更新或流式处理。|
|**正在删除**| 正在删除直播活动。 此暂时性状态下不会发生计费。 此状态下不允许进行更新或流式处理。|

## <a name="next-steps"></a>后续步骤

- [实时传送视频流概述](live-streaming-overview.md)
- [实时传送视频流教程](stream-live-tutorial-with-api.md)
