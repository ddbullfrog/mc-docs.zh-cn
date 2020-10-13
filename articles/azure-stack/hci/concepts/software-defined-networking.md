---
title: Azure Stack HCI 中的软件定义的网络 (SDN)
description: 适用于 Azure Stack HCI 中功能的 SDN 主题的概述。
author: WenJason
ms.author: v-jay
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
origin.date: 09/24/2020
ms.date: 10/12/2020
ms.openlocfilehash: 89b4257e9ed724c750b522a9e4f5b681ffad0a35
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451253"
---
# <a name="sdn-in-azure-stack-hci"></a>Azure Stack HCI 中的 SDN

> 适用于 Azure Stack HCI 版本 20H2；Windows Server 2019

软件定义的网络 (SDN) 提供了一种方法，用于在数据中心中集中配置和管理网络和网络服务，如交换、路由和负载均衡。 可以使用 SDN 动态创建、保护和连接网络，以满足不断演变的应用需求。 用于服务（如 Azure）的全球规模数据中心网络每天会高效地执行数以万计的网络更改，只能通过 SDN 来实现运营。

虚拟网络元素（例如 [Hyper-V 虚拟交换机](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch)、[Hyper-V 网络虚拟化](https://docs.microsoft.com/windows-server/networking/sdn/technologies/hyper-v-network-virtualization/hyper-v-network-virtualization)、[软件负载均衡](https://docs.microsoft.com/windows-server/networking/sdn/technologies/network-function-virtualization/software-load-balancing-for-sdn)和 [RAS 网关](https://docs.microsoft.com/windows-server/networking/sdn/technologies/network-function-virtualization/ras-gateway-for-sdn)）的作用是充当 SDN 基础结构的构成部分。 还可以使用现有 SDN 兼容设备，在虚拟网络中运行的工作负载与物理网络之间实现更深入的集成。

Azure Stack HCI 上有三个主要 SDN 组件，你可以选择要部署的组件：网络控制器、软件负载均衡器和网关。

## <a name="network-controller"></a>网络控制器

[网络控制器](https://docs.microsoft.com/windows-server/networking/sdn/technologies/Software-Defined-Networking-Technologies#network-controller)提供一种集中的可编程自动操作点，用于对数据中心的虚拟网络基础结构进行管理、配置、监视和故障排除。 它是高度可缩放的服务器角色，使用 Service Fabric 提供高可用性。 网络控制器必须部署在其自己的专用 VM 上。

部署网络控制器可实现以下功能：

- 创建和管理虚拟网络和子网。 将虚拟机 (VM) 连接到虚拟子网。
- 为连接到虚拟网络或基于 VLAN 的传统网络的 VM 配置和管理微分段。
- 将虚拟设备连接到虚拟网络。
- 为连接到虚拟网络或基于 VLAN 的传统网络的 VM 配置服务质量 (QoS) 策略。

建议在创建 Azure Stack HCI 群集之后，[使用 PowerShell 部署网络控制器](../deploy/network-controller-powershell.md)。

## <a name="software-load-balancing"></a>软件负载均衡 (SLB)

[软件负载均衡](https://docs.microsoft.com/windows-server/networking/sdn/technologies/network-function-virtualization/software-load-balancing-for-sdn) (SLB) 可用于在多个 VM 之间均匀分布客户网络流量。 它使多台服务器可以托管相同的工作负载，从而提供高可用性和可伸缩性。 SLB 使用[边界网关协议](https://docs.microsoft.com/windows-server/remote/remote-access/bgp/border-gateway-protocol-bgp)向物理网络播发虚拟 IP 地址。

## <a name="gateway"></a>网关

网关用于在虚拟网络与另一个网络（本地或远程）之间路由网络流量。 网关可用于：

- 通过 Internet 在 SDN 虚拟网络与外部客户网络之间创建安全的站点到站点 IPsec 连接。
- 在 SDN 虚拟网络与外部网络之间创建通用路由封装 (GRE) 连接。 站点到站点连接与 GRE 连接的不同之处在于后者不是加密连接。 有关 GRE 连接方案的详细信息，请参阅 [Windows Server 中的 GRE 隧道](https://docs.microsoft.com/windows-server/remote/remote-access/ras-gateway/gre-tunneling-windows-server)。
- 在 SDN 虚拟网络与外部网络之间创建第 3 层连接。 在这种情况下，SDN 网关只充当虚拟网络与外部网络之间的路由器。

网关使用[边界网关协议](https://docs.microsoft.com/windows-server/remote/remote-access/bgp/border-gateway-protocol-bgp)播发 GRE 终结点，并建立点到点连接。 SDN 部署会创建支持所有连接类型的默认网关池。 在此池中，可以指定保留为备用以防活动网关出现故障的网关数。

## <a name="next-steps"></a>后续步骤

如需相关信息，另请参阅：

- [规划软件定义的网络基础结构](plan-software-defined-networking-infrastructure.md)
- [Windows Server 中的 SDN 概述](https://docs.microsoft.com/windows-server/networking/sdn/software-defined-networking)
- [使用脚本部署软件定义的网络基础结构](https://docs.microsoft.com/windows-server/networking/sdn/deploy/deploy-a-software-defined-network-infrastructure-using-scripts)
