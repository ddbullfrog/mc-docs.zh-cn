---
title: 使用 Azure 门户缩放流式处理终结点 | Microsoft Docs
description: 本教程逐步演示如何使用 Azure 门户缩放流式处理终结点。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.assetid: 1008b3a3-2fa1-4146-85bd-2cf43cd1e00e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 03/19/2019
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: caba7c39f295236294f53156cb9ed95086a3db47
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91244869"
---
# <a name="scale-streaming-endpoints-with-the-azure-portal"></a>使用 Azure 门户缩放流式处理终结点

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>概述

> [!NOTE]
> 要完成本教程，需要一个 Azure 帐户。 有关详细信息，请参阅 [Azure 试用](https://www.azure.cn/pricing/1rmb-trial/)。
> 
> 

高级  流式处理终结点适合用于高级工作负载，同时提供可缩放的专用带宽容量。 默认情况下，使用高级  流式处理终结点的客户会获得一个流式处理单位 (SU)。 可通过添加 SU 来缩放流式处理终结点。 每个 SU 为应用程序提供额外的带宽容量。 有关流式处理终结点类型的详细信息，请参阅[流式处理终结点概述](media-services-streaming-endpoints-overview.md)主题。
 
本主题介绍了如何缩放流式处理终结点。

有关定价详细信息，请参阅[媒体服务定价详细信息](https://go.microsoft.com/fwlink/?LinkId=275107)。

## <a name="scale-streaming-endpoints"></a>缩放流式处理终结点

若要更改流式处理单位数，请按照以下步骤操作：

1. 在 [Azure 门户](https://portal.azure.cn/)中，选择 Azure 媒体服务帐户。
2. 在“设置”  窗口中，选择“流式处理终结点”  。
3. 单击要缩放的流式处理终结点。 

    > [!NOTE] 
    > 只能缩放高级  流式处理终结点。

4. 滑动滑块，以指定流式处理单位数。

    ![流式处理终结点](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints3.png)

## <a name="next-steps"></a>后续步骤
查看媒体服务学习路径。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

