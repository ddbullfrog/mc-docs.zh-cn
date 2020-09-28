---
title: Azure 虚拟 WAN 合作伙伴和位置 | Azure
description: 本文包含 Azure 虚拟 WAN 的合作伙伴和中心位置的列表。
services: virtual-wan
author: rockboyfor
ms.service: virtual-wan
ms.topic: conceptual
origin.date: 02/12/2020
ms.date: 09/25/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
Customer intent: As someone with a networking background, I want to find a Virtual WAN partner
ms.openlocfilehash: c116a307ca00516f1f0923e7f075b595ffe41a36
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246710"
---
# <a name="virtual-wan-partners-and-virtual-hub-locations"></a>虚拟 WAN 合作伙伴和虚拟中心位置

本文提供有关虚拟 WAN 针对连接到虚拟中心所支持的区域以及合作伙伴的信息。

Azure Virtual WAN 是一种网络服务，它通过 Azure 提供经优化、自动的分支到分支连接。 使用虚拟 WAN 可以连接分支设备，并将其配置为与 Azure 通信。 可以手动完成此操作，也可以通过虚拟 WAN 合作伙伴使用提供商设备完成此操作。 使用合作伙伴设备可以降低操作难度、简化连接和进行配置管理。

本地设备到虚拟中心的连接将自动建立。 虚拟中心是 Azure 托管的虚拟网络。 中心包含不同的服务终结点，可从本地网络 (vpnsite) 建立连接。 每个区域只能有一个中心。

<!--CORRECT ON an Azure managed-->

<a name="automation"></a>
## <a name="automation-from-connectivity-partners"></a>连接合作伙伴提供的自动化

连接到 Azure 虚拟 WAN 的设备具有内置的自动化连接功能。 这通常在设备管理 UI（或同等位置）中设置，该 UI 设置 VPN 分支设备到 Azure 虚拟中心 VPN 终结点（VPN 网关）之间的连接和配置管理。

以下高级自动化在设备控制台/管理中心中设置：

* 设备访问 Azure 虚拟 WAN 资源组的相应权限
* 将分支设备上传到 Azure 虚拟 WAN
* 自动下载 Azure 连接信息
* 本地分支设备的配置 

某些连接合作伙伴可以扩展自动化，包括创建 Azure 虚拟中心 VNet 和 VPN 网关。 若要了解有关自动化的详细信息，请参阅[适用于虚拟 WAN 合作伙伴的自动化指南](virtual-wan-configure-automation-providers.md)。

<a name="partners"></a>
## <a name="connectivity-through-partners"></a>通过合作伙伴建立连接

[!INCLUDE [partners](../../includes/virtual-wan-partners-include.md)]

在不久的将来，我们的路线图中将列出以下合作伙伴：128 Technologies、Arista、Aruba HPE、Cisco Systems (Viptela)、F5 Networks、Open Systems、Oracle SD-WAN 和 SharpLink。

<a name="locations"></a>
## <a name="locations"></a>位置

[!INCLUDE [regions](../../includes/virtual-wan-regions-include.md)]

## <a name="next-steps"></a>后续步骤

* 有关虚拟 WAN 的详细信息，请参阅[虚拟 WAN 常见问题解答](virtual-wan-faq.md)。

* 有关如何自动连接到 Azure 虚拟 WAN 的详细信息，请参阅[适用于虚拟 WAN 合作伙伴的自动化指南](virtual-wan-configure-automation-providers.md)。

<!-- Update_Description: update meta properties, wording update, update link -->