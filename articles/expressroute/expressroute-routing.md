---
title: Azure ExpressRoute：路由要求
description: 本页提供有关为 ExpressRoute 线路配置和管理路由的详细要求。
services: expressroute
author: ganesr
ms.service: expressroute
ms.topic: conceptual
origin.date: 09/19/2019
ms.author: v-yiso
ms.date: 12/02/2019
ms.openlocfilehash: a94467597e3e710fa6bfbe6a602f2071ac73a0f6
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211420"
---
# <a name="expressroute-routing-requirements"></a>ExpressRoute 路由要求
若要使用 ExpressRoute 连接到 Microsoft 云服务，需要设置并管理路由。 某些连接服务提供商以托管服务形式提供路由的设置和管理。 请咨询连接服务提供商，以确定他们是否提供此类服务。 如果不提供，则必须遵守以下要求：

有关需要在设置后才能建立连接的路由会话的说明，请参阅[线路和路由域](./expressroute-circuit-peerings.md)一文。

> [!NOTE]
> Microsoft 不支持将任何路由器冗余协议（例如 HSRP 和 VRRP）用于高可用性配置。 我们依赖每个对等互连的一组冗余 BGP 会话来获得高可用性。
> 
> 

## <a name="ip-addresses-used-for-peerings"></a>用于对等互连的 IP 地址
需要保留一些 IP 地址块用于配置网络与 Microsoft Enterprise Edge (MSEE) 路由器之间的路由。 本部分提供了要求列表，并介绍有关如何获取和使用这些 IP 地址的规则。

### <a name="ip-addresses-used-for-azure-private-peering"></a>用于 Azure 专用对等互连的 IP 地址
可以使用专用 IP 地址或公共 IP 地址配置对等互连。 用于配置路由的地址范围不得与用于在 Azure 中创建虚拟网络的地址范围重叠。 

* 必须为路由接口保留一个 /29 子网或两个 /30 子网。
* 用于路由的子网可以是专用 IP 地址或公共 IP 地址。
* 子网不得与客户保留用于 Microsoft 云的范围冲突。
* 如果使用 /29 子网，它将拆分成两个 /30 子网。 
  * 第一个 /30 子网用于主链路，第二个 /30 子网用于辅助链路。
  * 对于每个 /30 子网，必须在路由器上使用 /30 子网的第一个 IP 地址。 Microsoft 使用 /30 子网的第二个 IP 地址设置 BGP 会话。
  * [可用性 SLA](https://www.azure.cn/support/legal/sla/) 只有在设置两个 BGP 会话后才有效。  

#### <a name="example-for-private-peering"></a>专用对等互连示例
如果选择使用 a.b.c.d/29 设置对等互连，它将拆分成两个 /30 子网。 在以下示例中，请注意 a.b.c.d/29 子网的用法：

* a.b.c.d/29 拆分成 a.b.c.d/30 和 a.b.c.d+4/30 并通过预配 API 一路传递到 Microsoft。
  * 请使用 a.b.c.d+1 作为主要 PE 的 VRF IP，而 Microsoft 将使用 a.b.c.d+2 作为主要 MSEE 的 VRF IP。
  * 请使用 b.c.d+5 作为辅助 PE 的 VRF IP，而 Microsoft 将使用 a.b.c.d+6 作为辅助 MSEE 的 VRF IP。

假设选择 192.168.100.128/29 设置专用对等互连。 192.168.100.128/29 包括从 192.168.100.128 到 192.168.100.135 的地址，其中：

- 192.168.100.128/30 分配给 link1（提供商使用 192.168.100.129，Microsoft 使用 192.168.100.130）。
- 192.168.100.132/30 分配给 link2（提供商使用 192.168.100.133，Microsoft 使用 192.168.100.134）。

### <a name="ip-addresses-used-for-microsoft-peering"></a>用于 Microsoft 对等互连的 IP 地址
必须使用自己的公共 IP 地址设置 BGP 会话。 Microsoft 必须能够通过路由 Internet 注册表和 Internet 路由注册表验证 IP 地址的所有权。

* 门户中列出的与 Microsoft 对等互连的已播发公共前缀相对应的 IP 将为 Microsoft 核心路由器创建 ACL，目的是允许来自这些 IP 的入站流量。 
* 必须使用一个唯一的 /29 (IPv4) 或 /125 (IPv6) 子网或两个 /30 (IPv4) 或 /126 (IPv6) 子网为每条 ExpressRoute 线路（如果有多个）的每个对等互连设置 BGP 对等互连。
* 如果使用 /29 子网，它将拆分成两个 /30 子网。
* 第一个 /30 子网用于主链路，第二个 /30 子网将用于辅助链路。
* 对于每个 /30 子网，必须在路由器上使用 /30 子网的第一个 IP 地址。 Microsoft 使用 /30 子网的第二个 IP 地址设置 BGP 会话。
* 如果使用 /125 子网，它将拆分成两个 /126 子网。
* 第一个 /126 子网用于主链路，第二个 /126 子网将用于辅助链路。
* 对于每个 /126 子网，必须在路由器上使用 /126 子网的第一个 IP 地址。 Microsoft 使用 /126 子网的第二个 IP 地址设置 BGP 会话。
* 只有设置两个 BGP 会话，我们的 [可用性 SLA](https://www.azure.cn/support/legal/sla/) 才有效。

### <a name="ip-addresses-used-for-azure-public-peering"></a>用于 Azure 公共对等互连的 IP 地址

> [!NOTE]
> Azure 公共对等互连不适用于新线路。
> 

必须使用自己的公共 IP 地址设置 BGP 会话。 Microsoft 必须能够通过路由 Internet 注册表和 Internet 路由注册表验证 IP 地址的所有权。 

* 必须使用一个唯一的 /29 子网或两个 /30 子网为每条 ExpressRoute 线路（如果有多个）的每个对等互连设置 BGP 对等互连。 
* 如果使用 /29 子网，它将拆分成两个 /30 子网。 
  * 第一个 /30 子网用于主链路，第二个 /30 子网用于辅助链路。
  * 对于每个 /30 子网，必须在路由器上使用 /30 子网的第一个 IP 地址。 Microsoft 使用 /30 子网的第二个 IP 地址设置 BGP 会话。
  * 只有设置两个 BGP 会话，我们的 [可用性 SLA](https://azure.microsoft.com/support/legal/sla/) 才有效。

## <a name="public-ip-address-requirement"></a>公共 IP 地址要求

### <a name="private-peering"></a>专用对等互连
可选择使用用于专用对等互连的公共或专用 IPv4 地址。 我们会对用户的流量进行端到端隔离，因此在进行专用对等互连时，不可能出现与其他客户的地址发生重叠的情况。 这些地址不会播发到 Internet。 

### <a name="microsoft-peering"></a>Microsoft 对等互连
可以使用 Microsoft 对等互连路径连接到 Microsoft 云服务。 服务列表包括 Office 365 服务，例如 Exchange Online、SharePoint Online、Skype for Business 和 Microsoft Teams。 Microsoft 在 Microsoft 对等互连上支持双向连接。 定向到 Microsoft 云服务的流量必须使用有效的公共 IPv4 地址才能进入 Microsoft 网络。

确保 IP 地址和 AS 号码已在下列其中一个注册表中注册：

* [ARIN](https://www.arin.net/)
* [APNIC](https://www.apnic.net/)
* [AFRINIC](https://www.afrinic.net/)
* [LACNIC](https://www.lacnic.net/)
* [RIPENCC](https://www.ripe.net/)
* [RADB](https://www.radb.net/)
* [ALTDB](https://altdb.net/)

如果没有在前述注册表中为你分配前缀和 AS 编号，需开立一个支持案例，以便手动验证前缀和 ASN。 支持需要文档，例如证明你有权使用相关资源的授权书。

专用 AS 编号可以用于 Microsoft 对等互连，但也需手动验证。 此外，对于收到的前缀，我们会删除 AS PATH 中的专用 AS 数字。 因此，无法在 AS PATH 中追加专用 AS 数字来[影响 Microsoft 对等互连的路由](expressroute-optimize-routing.md)。 

> [!IMPORTANT]
> 不要将相同的公共 IP 路由播发到公共 Internet 和通过 ExpressRoute 播发。 为了降低错误配置导致不对称路由的风险，我们强烈建议通过 ExpressRoute 播发到 Microsoft 的 NAT IP 地址应该来自完全没有播发到 Internet 的范围。 如果无法实现这一点，则必须确保通过 ExpressRoute 播发的范围比 Internet 连接上的范围更具体。 除了要进行 NAT 的公共路由外，还可以在本地网络中通过 ExpressRoute 播发与 Microsoft 中的 Office 365 终结点通信的服务器使用的公共 IP 地址。 
> 
> 

### <a name="public-peering-deprecated---not-available-for-new-circuits"></a>公共对等互连（已弃用 - 不适用于新线路）
Azure 公共对等互连路径使用户能够通过其公共 IP 地址连接到 Azure 中托管的所有服务。 这些服务包括 [ExpessRoute 常见问题](expressroute-faqs.md) 中列出的服务，以及由 ISV 托管在 Microsoft Azure 上的所有服务。 始终从用户网络向 Microsoft 网络发起与公共对等互连中 Microsoft Azure 服务的连接。 必须为定向到 Microsoft 网络的流量使用公共 IP 地址。

> [!IMPORTANT]
> 所有 Azure PaaS 服务可通过 Microsoft 对等互连访问。
>   

公共对等互连允许使用专用 AS 编号。

## <a name="dynamic-route-exchange"></a>动态路由交换
路由交换会通过 eBGP 协议进行。 在 MSEE 与路由器之间建立 EBGP 会话。 不要求对 BGP 会话进行身份验证。 如果需要，可以配置 MD5 哈希。 有关配置 BGP 会话的信息，请参阅[配置路由](how-to-routefilter-portal.md)及[线路预配工作流和线路状态](expressroute-workflows.md)。

## <a name="autonomous-system-numbers"></a>自治系统编号
Microsoft 使用 AS 12076 进行 Azure 公共、Azure 专用和 Microsoft 对等互连。 我们保留了 ASN 65515-65520 供内部使用。 支持 16 和 32 位 AS 编号。

数据传输对称没有相关要求。 转发与返回路径可以遍历不同的路由器对。 相同的路由必须从属于你的多个线路对的任何一端播发。 路由指标不需要完全相同。

## <a name="route-aggregation-and-prefix-limits"></a>路由聚合与前缀限制
支持通过 Azure 专用对等互连播发最多 4000 个前缀。 如果启用 ExpressRoute 高级外接程序，则最多可增加 10,000 个前缀。 接受为每个 BGP 会话最多使用 200 个前缀建立 Azure 公共和 Microsoft 对等互连。 

如果前缀数目超过此限制，将丢弃 BGP 会话。 只接受专用对等互连链路上的默认路由。 提供商必须从 Azure 公共和 Microsoft 对等互连路径中筛选默认路由和专用 IP 地址 (RFC 1918)。 

## <a name="transit-routing-and-cross-region-routing"></a>传输路由和跨区域路由
ExpressRoute 不能配置为传输路由器。 必须依赖连接服务提供商的传输路由服务。

## <a name="advertising-default-routes"></a>播发默认路由
只有 Azure 专用对等互连会话允许默认路由。 在这种情况下，可将所有流量从关联的虚拟网络路由到用户网络。 在专用对等互连中播发默认路由会导致阻止来自 Azure 的 Internet 路径。 必须依赖企业网络边缘，为 Azure 中托管的服务往返路由 Internet 的流量。 

 若要启用与其他 Azure 服务和基础结构服务的连接，必须确保已准备下列其中一项：

* 已启用 Azure 公共对等互连，以将流量路由到公共终结点。
* 使用用户定义的路由可为需要 Internet 连接的每个子网建立 Internet 连接。

> [!NOTE]
> 播发默认路由会中断 Windows 和其他 VM 许可证激活。 请按照 [此处](https://blogs.msdn.com/b/mast/archive/2015/05/20/use-azure-custom-routes-to-enable-kms-activation-with-forced-tunneling.aspx) 的说明来解决此问题。
> 
> 

## <a name="next-steps"></a>后续步骤
* 配置 ExpressRoute 连接。
  
  * [创建和修改线路](expressroute-howto-circuit-arm.md)
  * [创建和修改对等互连配置](expressroute-howto-routing-arm.md)
  * [将 VNet 链接到 ExpressRoute 线路](expressroute-howto-linkvnet-arm.md)