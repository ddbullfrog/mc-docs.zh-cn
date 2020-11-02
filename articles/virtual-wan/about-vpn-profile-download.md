---
title: Azure 虚拟 WAN - 用户 VPN 客户端配置文件
description: 这可帮助你使用客户端配置文件
services: virtual-wan
ms.service: virtual-wan
ms.topic: how-to
origin.date: 09/22/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: f9e38c944816ec5932461e18205159a14a030909
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103966"
---
<!--Verified successfully on VPN client -->
# <a name="working-with-user-vpn-client-profiles"></a>使用用户 VPN 客户端配置文件

已下载的配置文件包含配置 VPN 连接所需的信息。 本文可帮助你获取和了解用户 VPN 客户端配置文件所需的信息。

[!INCLUDE [client profiles](../../includes/vpn-gateway-vwan-vpn-profile-download.md)]

* OpenVPN 文件夹包含 ovpn 配置文件，需要对该文件进行修改以包含密钥和证书。 有关详细信息，请参阅[配置 OpenVPN 客户端](../virtual-wan/howto-openvpn-clients.md#windows)。

## <a name="next-steps"></a>后续步骤

有关虚拟 WAN 用户 VPN 的详细信息，请参阅[创建用户 VPN 连接](virtual-wan-point-to-site-portal.md)。

<!-- Update_Description: update meta properties, wording update, update link -->