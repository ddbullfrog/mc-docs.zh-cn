---
title: 下载 Azure 虚拟 WAN 全局或基于中心的 VPN 配置文件 | Azure
description: 了解有关虚拟 WAN 自动化可缩放的分支到分支连接、可用区域和合作伙伴的信息。
services: virtual-wan
ms.service: virtual-wan
ms.topic: how-to
origin.date: 04/20/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: yes
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.openlocfilehash: 379cc91b983f2805dc69d0e2fd18c4766c53bfec
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246828"
---
# <a name="download-a-global-or-hub-based-profile-for-user-vpn-clients"></a>为用户 VPN 客户端下载全局或基于中心的配置文件

Azure 虚拟 WAN 为远程用户提供两种类型的连接：全局连接和基于中心的连接。 使用以下部分了解和下载配置文件。 

> [!IMPORTANT]
> RADIUS 身份验证仅支持基于中心的配置文件。

## <a name="global-profile"></a>全局配置文件

该配置文件指向包含所有活动用户 VPN 中心的负载均衡器。 用户将被定向到最靠近用户地理位置的中心。 当用户经常前往不同的位置时，这种类型的连接非常有用。 若要下载**全局**配置文件，请执行以下操作：

1. 导航到虚拟 WAN。
2. 单击“用户 VPN 配置”****。
3. 突出显示要下载其配置文件的配置。
4. 单击“下载虚拟 WAN 用户 VPN 配置文件”****。

    :::image type="content" source="./media/global-hub-profile/global1.png" alt-text="全局配置文件":::

## <a name="hub-based-profile"></a>基于中心的配置文件

配置文件指向单个中心。 用户只能使用此配置文件连接到特定中心。 若要下载**基于中心**的配置文件，请执行以下操作：

1. 导航到虚拟 WAN。
2. 在“概述”页中单击“中心”****。

    :::image type="content" source="./media/global-hub-profile/hub1.png" alt-text="中心配置文件 1":::
3. 单击“用户 VPN (点到站点)”****。
4. 单击“下载虚拟中心用户 VPN 配置文件”****。

    :::image type="content" source="./media/global-hub-profile/hub2.png" alt-text="中心配置文件 2":::
5. 选中“EAPTLS”****。
6. 单击“生成并下载配置文件”****。

    :::image type="content" source="./media/global-hub-profile/download.png" alt-text="中心配置文件 3":::

## <a name="next-steps"></a>后续步骤

若要详细了解虚拟 WAN，请参阅[虚拟 WAN 概述](virtual-wan-about.md)页。

<!-- Update_Description: update meta properties, wording update, update link -->