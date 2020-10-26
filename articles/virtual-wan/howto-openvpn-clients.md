---
title: 为 Azure 虚拟 WAN 配置 OpenVPN 客户端
description: 为 Azure 虚拟 WAN 配置 OpenVPN 客户端的步骤
services: virtual-wan
ms.service: virtual-wan
ms.topic: how-to
origin.date: 09/22/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.openlocfilehash: 8621360898d0679bd4f9b77ddaeb960241a8a6e5
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472673"
---
<!--Verified successfully on VPN OpenVPN client-->
# <a name="configure-an-openvpn-client-for-azure-virtual-wan"></a>为 Azure 虚拟 WAN 配置 OpenVPN 客户端

本文可帮助你配置 OpenVPN &reg; 协议  客户端。

## <a name="before-you-begin"></a>准备阶段

创建用户 VPN（点到站点）配置。 请确保选择“OpenVPN”作为隧道类型。 有关步骤，请参阅[为 Azure 虚拟 WAN 创建 P2S 配置](virtual-wan-point-to-site-portal.md#p2sconfig)。

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>后续步骤

有关用户 VPN（点到站点）的详细信息，请参阅[创建用户 VPN 连接](virtual-wan-point-to-site-portal.md)。

**“OpenVPN”是 OpenVPN Inc. 的商标。**

<!-- Update_Description: update meta properties, wording update, update link -->