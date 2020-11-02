---
title: 使用 Azure 虚拟 WAN 和安全中心与中国互连
description: 了解如何使用 Azure 虚拟 WAN 和安全中心与中国互连
services: virtual-wan
ms.service: virtual-wan
ms.topic: conceptual
origin.date: 09/22/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: a75a5754811173af1180fd98948ba4d19b4b0539
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103961"
---
<!--Verified successfully-->
# <a name="interconnect-with-china-using-azure-virtual-wan-and-secure-hub"></a>使用 Azure 虚拟 WAN 和安全中心与中国互连

在普通的汽车、制造、物流行业或大使馆等机构中，经常问到的一个问题是如何改进与中国的互连。 这些改进措施主要与使用云服务（例如 Office 365、Azure 全球服务）或通过客户主干网络使用中国境内的互连分支相关。

在大多数情况下，连接到中国境外（例如欧洲或美国）的客户都会挣扎于高延迟、低带宽、不稳定连接以及高费用的问题。

这些阻碍的原因在于“中国防火长城（中国国家防火墙）”，它会保护 Internet 上的中国网段部分，并筛选发往中国的流量。 从中国大陆发往中国境外（香港和澳门等特别行政区除外）的几乎所有流量都要流经防火长城。 流经香港和澳门的流量不会强力进入防火长城，而是由防火长城的一部分机制进行处理。

:::image type="content" source="./media/interconnect-china/provider.png" alt-text="提供商互连":::

使用虚拟 WAN，客户可与 Azure 云服务建立性能更高且更稳定的连接，并可连接到其企业网络，同时不会违反中国网络安全法。

<a name="requirements"></a>
## <a name="requirements-and-workflow"></a>要求和工作流

若要持续遵守中国网络安全法，需要满足一组特定的条件。

首先，需要与拥有中国 ICP（Internet 内容提供商）执照的 ISP 合作。 在大多数情况下，你最终会与以下提供商之一合作：

* 中国电信国际有限公司
* 中国移动有限公司
* 中国联通有限公司
* PCCW Global Ltd.（香港电讯盈科环球有限公司）
* 香港电信有限公司

根据提供商以及你的需求，你目前需要购买以下网络连接服务之一才能与中国境内的分支建立互连。

* MPLS/IPVPN 网络 
* 软件定义的 WAN (SDWAN)
* 专用 Internet 访问

接下来，需要同意该提供商在 Azure 全球网络及其位于香港（而不是北京或上海）的边缘网络中提供一个接入点。 在这种情况下，香港由于其自身的物理连接优势以及在中国的归属而显得非常重要。

大多数客户觉得新加坡在地图上与中国较为靠近，因此最适合在新加坡建立互连，但事实上并非如此。 观察网络光纤地图时可以发现，与中国的几乎所有网络连接都是通过北京、上海和香港建立的。 因此，香港是更适合与中国建立互连的位置。

根据具体的提供商，你可能会获得不同的服务产品。 下表根据撰写本文时的信息，列出了提供商的示例及其提供的服务。

| 服务 | 提供商示例 |
| --- | --- |
| MPLS/IPVPN 网络 |PCCW、中国电信国际有限公司 |
|SDWAN| PCCW、中国电信国际有限公司|
| 专用 Internet 访问 | PCCW、香港电信、中国移动|

可以与提供商议定要使用以下两种解决方案中的哪一种来接入 Azure 全球主干网：

* 获取在香港终止流量的 Azure ExpressRoute。 使用 MPLS/IPVPN 时需要采用这种解决方案。 目前，能够通过 ExpressRoute 连接到香港且持有 ICP 执照的唯一一家提供商是中国电信国际有限公司。 但是，提供商还可以通过 Megaport 或 InterCloud 等云交换提供商与其他提供商通信。 有关详细信息，请参阅 [ExpressRoute 连接提供商](../expressroute/expressroute-locations-providers.md#partners)。

* 在以下 Internet 交换点之一或通过专用网络互连直接使用专用 Internet 访问。

以下列表显示了香港提供的 Internet 交换点：

* AMS-IX 香港
* BBIX 香港
* Equinix 香港
* HKIX

使用此连接方式时，Azure 服务的下一 BGP 跃点必须是 Azure 自治系统编号 (AS#) 8075。 如果使用单一位置或 SDWAN 解决方案，则会选择该解决方案建立连接。

无论采用哪种方式，我们仍建议在中国大陆建立另一个常规 Internet 接入点。 其目的是为了在发往云服务（例如 Microsoft 365 和 Azure）的企业流量以及法律管制的 Internet 流量之间进行划分。

中国境内合规的网络体系结构如以下示例所示：

:::image type="content" source="./media/interconnect-china/multi-branch.png" alt-text="提供商互连":::

在此示例中，与位于香港的 Azure 全球网络建立互连后，可以开始利用 [Azure 虚拟 WAN 全球传输体系结构](virtual-wan-global-transit-network-architecture.md)和其他服务（例如 Azure 安全虚拟 WAN 中心）来使用服务，并互连到中国境外的分支和数据中心。

<a name="hub-to-hub"></a>
## <a name="hub-to-hub-communication"></a>中心到中心通信

在本部分，我们将使用虚拟 WAN 中心到中心通信进行互连。 在此方案中，你将创建一个新的虚拟 WAN 中心资源，以连接到位于香港、首选的其他区域、已包含 Azure 资源的区域或你要连接到的位置的虚拟 WAN 中心。

示例体系结构如以下示例所示：

:::image type="content" source="./media/interconnect-china/sample.png" alt-text="提供商互连":::

在此示例中，中国分支使用 VPN 或 MPLS 连接连接到 Azure 中国云并相互连接。 需要连接到全球服务的分支使用直接连接到香港的 MPLS 或基于 Internet 的服务。 

<!--Not Available on  [ExpressRoute Global Reach](../expressroute/expressroute-global-reach.md)-->

ExpressRoute Global Reach 在某些区域不可用。 例如，如果需要与巴西或印度互连，则需利用[云交换提供商](../expressroute/expressroute-locations.md)来提供路由服务。

<!--Not Available on #connectivity-through-exchange-providers-->

下图演示了此方案的上述两种示例连接。

:::image type="content" source="./media/interconnect-china/global.png" alt-text="提供商互连":::

<a name="secure"></a>
## <a name="secure-internet-breakout-for-office-365"></a>适用于 Office 365 的安全 Internet 接入点

另一个注意事项是网络安全性，以及中国和虚拟 WAN 建立的主干组件与客户主干网之间的入口点的日志记录。 在大多数情况下，需要通过接入点连接到香港的 Internet 才能直接访问 Microsoft 边缘网络，为此，Microsoft 365 服务会使用 Azure Front Door 服务器。

<!--Not Available on [Azure Virtual WAN secured hub](../firewall-manager/secured-virtual-hub.md)-->

下图演示了此方案的示例：

:::image type="content" source="./media/interconnect-china/internet.png" alt-text="提供商互连":::

<a name="traffic"></a>
## <a name="architecture-and-traffic-flows"></a>体系结构和流量流

整体体系结构可能略有不同，具体取决于你选择如何与香港建立连接。 本部分介绍采用不同 VPN 或 SDWAN 和/或 ExpressRoute 组合的三种可用体系结构。

所有这些选项都利用由 Azure 虚拟 WAN 保护的中心在香港建立直接的 Microsoft 365 连接。 这些体系结构还支持 [Microsoft 365 多地域](https://docs.microsoft.com/microsoft-365/enterprise/microsoft-365-multi-geo)的合规要求，并使该流量靠近下一个 Office 365 Front Door 位置。 因此，它也会改善中国境外的 Microsoft 365 使用情况。

<!--CORRECT ON https://docs.microsoft.com/microsoft-365/enterprise/microsoft-365-multi-geo-->

<!--Not Available on [21Vianet Azure Peering Services (MAPS)](https://docs.microsoft.com/azure/peering-service/about)-->

<a name="option-1"></a>
### <a name="option-1-sdwan-or-vpn"></a>选项 1：SDWAN 或 VPN

本部分讨论了使用 SDWAN 或 VPN 连接到香港和其他分支的设计。 此选项展示了在虚拟 WAN 主干网络的两个站点上使用单纯 Internet 连接时的用法和流量流。 在这种情况下，将使用专用 Internet 访问或 ICP 提供商 SDWAN 解决方案连接到香港。 其他分支也使用单纯 Internet 或 SDWAN 解决方案。

:::image type="content" source="./media/interconnect-china/china-traffic.png" alt-text="提供商互连":::

在此体系结构中，每个站点都使用 VPN 和 Azure 虚拟 WAN 连接到 Azure 全球网络。 站点与香港之间的流量通过 Azure 网络传输，只在最后一英里范围内才使用常规的 Internet 连接。

<a name="option-2"></a>
### <a name="option-2-expressroute-and-sdwan-or-vpn"></a>选项 2：ExpressRoute 和 SDWAN 或 VPN

本部分讨论了在香港使用 ExpressRoute 并对其他分支使用 VPN/SDWAN 的设计。 此选项展示了如何使用在香港以及通过 SDWAN 或 VPN 连接的其他分支中终止的 ExpressRoute。 香港的 ExpressRoute 目前限制为少量的提供商，可以在 [Express Route 合作伙伴](../expressroute/expressroute-locations-providers.md)列表中找到这些提供商。

<!--Not Available on #global-commercial-azure-->

:::image type="content" source="./media/interconnect-china/expressroute.png" alt-text="提供商互连":::

还可以通过其他一些选项从中国终止 ExpressRoute，例如，在韩国或日本终止。 但出于合规性、管制和延迟方面的原因，香港目前是最佳选择。

<a name="option-3"></a>
### <a name="option-3-expressroute-only"></a>选项 3：仅使用 ExpressRoute

本部分讨论了在香港和其他分支中使用 ExpressRoute 的设计。 此选项展示了如何在两端使用 ExpressRoute 建立互连。 此处的流量流与其他选项不同。 Microsoft 365 流量将流向 Azure 虚拟 WAN 保护的中心，然后从该中心流向 Microsoft 边缘网络和 Internet。

发往互连分支或者从互连分支发往中国境内位置的流量在该体系结构中遵循不同的方法。 目前，虚拟 WAN 不支持 ExpressRoute 到 ExpressRoute 的传输。 流量将利用 ExpressRoute Global Reach 或第三方互连，而不会通过虚拟 WAN 中心。 它将直接从一个 Microsoft 企业边缘 (MSEE) 流向另一个 MSEE。

:::image type="content" source="./media/interconnect-china/expressroute-virtual.png" alt-text="提供商互连":::

目前，ExpressRoute Global Reach 并非在每个国家/地区都可用，但你可以使用 Azure 虚拟 WAN 来配置解决方案。

例如，可以使用 Azure 对等互连配置 ExpressRoute，并通过该对等互连将 VPN 隧道连接到 Azure 虚拟 WAN。 现在，你已再次实现了 VPN 与 ExpressRoute 之间的传输，且未使用 Global Reach 以及第三方提供商和服务（例如 Megaport Cloud）。

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅以下文章：

* [使用 Azure 虚拟 WAN 的全局传输网络体系结构](virtual-wan-global-transit-network-architecture.md)

* [创建虚拟 WAN 中心](virtual-wan-site-to-site-portal.md)

    <!--Not Available on * [Configure a Virtual WAN secured hub](../firewall-manager/secure-cloud-network.md)-->

    <!--Not Available on * [Azure Peering Service Preview Overview](https://docs.microsoft.com/azure/peering-service/about)-->

<!-- Update_Description: update meta properties, wording update, update link -->