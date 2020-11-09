---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/01/2020
title: 将 Azure Databricks 工作区连接到本地网络 - Azure Databricks
description: 了解如何将 Azure Databricks 工作区连接到本地网络。
ms.openlocfilehash: 0c13e176aa6a8092c03ecc02a00c5395983e1eca
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106482"
---
# <a name="connect-your-azure-databricks-workspace-to-your-on-premises-network"></a>将 Azure Databricks 工作区连接到本地网络

此概要指南介绍如何建立从 Azure Databricks 工作区到本地网络的连接。 它基于下图中显示的中心辐射型拓扑，其中流量通过传输虚拟网络 (VNet) 路由到本地网络。

此过程要求将 Azure Databricks 工作区[部署在你自己的虚拟网络](vnet-inject.md)中（也称为 VNet 注入）。

> [!div class="mx-imgBorder"]
> ![虚拟网络部署](../../../_static/images/account-settings/azure-networking-transit-vnet.png)

> [!NOTE]
>
> 请随时联系你的 Microsoft 和 Databricks 帐户团队，讨论本文中所述的配置过程。

## <a name="prerequisites"></a>先决条件

Azure Databricks 工作区必须部署在[你自己的虚拟网络](vnet-inject.md)中。

## <a name="step-1-set-up-a-transit-virtual-network-with-azure-virtual-network-gateway"></a>步骤 1：使用 Azure 虚拟网络网关设置传输虚拟网络

本地连接需要传输 VNet 中的虚拟网络网关（ExpressRoute 或 VPN）。 如果已存在，请跳到步骤 2。

* 如果已在本地网络和 Azure 之间设置了 ExpressRoute，请确保在传输 VNet 中设置虚拟网络网关，如[使用 Azure 门户为 ExpressRoute 配置虚拟网络网关](/expressroute/expressroute-howto-add-gateway-portal-resource-manager)中所述。
* 如果没有设置 ExpressRoute，请按照[使用 Azure 门户配置 VNet 到 VNet VPN 网关连接](/vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal)中的步骤 1-5 操作，创建具有基于 VPN 的虚拟网络网关的传输 VNet。

> [!NOTE]
>
> 若要获得帮助，请联系你的 Microsoft 帐户团队。

## <a name="step-2-peer-the-azure-databricks-virtual-network-with-the-transit-virtual-network"></a>步骤 2：将 Azure Databricks 虚拟网络和传输虚拟网络对等互连

如果 Azure Databricks 工作区与虚拟网络网关不在同一 VNet 中，请按照[对等互连虚拟网络](vnet-peering.md)中的说明将 Azure Databricks VNet 对等互连到传输 VNet，选择以下选项：

* 在 Azure Databricks VNet 端使用远程网关。
* 在传输 VNet 端允许网关传输。

可以在[创建对等互连](/virtual-network/virtual-network-manage-peering#create-a-peering)中了解有关这些选项的详细信息。

> [!NOTE]
>
> 如果与 Azure Databricks 的本地网络连接不能与上述设置一起使用，你还可以选择对等互连两侧的“允许转发的流量”选项来解决此问题。

有关为虚拟网络对等互连配置 VPN 网关传输的信息，请参阅[为虚拟网络对等互连配置 VPN 网关传输](/vpn-gateway/vpn-gateway-peering-gateway-transit)。

## <a name="step-3-create-user-defined-routes-and-associate-them-with-your-azure-databricks-virtual-network-subnets"></a><a id="create-routes"> </a><a id="step-3-create-user-defined-routes-and-associate-them-with-your-azure-databricks-virtual-network-subnets"> </a>步骤 3：创建用户定义的路由，并将其与 Azure Databricks 虚拟网络子网关联

一旦 Azure Databricks VNet 与传输 VNet 对等互连（使用虚拟网络网关），Azure 将通过传输 VNet 自动配置所有路由。 这可能会开始中断 Azure Databricks 工作区中的群集设置，因为可能缺少从群集节点到 Azure Databricks 控制平面的正确配置的返回路由。 因此，必须创建[用户定义的路由](/virtual-network/virtual-networks-udr-overview#custom-routes)（也称为 UDR 或自定义路由）。

1. 按照[创建路由表](/virtual-network/tutorial-create-route-table-portal#create-a-route-table)中的说明创建路由表。

   创建路由表时，请启用 BGP 路由传播。

   > [!NOTE]
   >
   > 如果在[测试](#on-prem-test)期间本地网络连接设置失败，你可能需要禁用 BGP 路由传播选项。 禁用仅作为最后的手段。

2. 使用[自定义路由](/virtual-network/virtual-networks-udr-overview#custom-routes)中的说明为以下服务添加用户定义的路由。

   | 源      | 地址前缀              | 下一跃点类型     |
   |-------------|-------------------------------|-------------------|
   | 默认     | 控制平面 NAT IP          | Internet          |
   | 默认     | Webapp IP                     | Internet          |
   | 默认     | 元存储 IP                  | Internet          |
   | 默认     | 项目 Blob 存储 IP      | Internet          |
   | 默认     | 日志 Blob 存储 IP           | Internet          |
   | 默认     | DBFS 根 Blob 存储 IP     | Internet          |

   若要获取这些服务中的每个服务的 IP 地址，请按照[用户定义的 Azure Databricks 路由设置](udr.md)中的说明进行操作。

3. 使用[将路由表关联到子网](/virtual-network/manage-route-table#associate-a-route-table-to-a-subnet)中的说明，将路由表与 Azure Databricks VNet 公共和专用子网关联。

   一旦自定义路由表与 Azure Databricks VNet 子网关联，就没有必要编辑网络安全组中的出站安全规则。 可以选择更改出站规则，使“Internet / Any”更具针对性，但这样做没有实际意义，因为路由将控制实际流出量。

> [!NOTE]
>
> 如果基于 IP 的路由在[测试](#on-prem-test)期间失败，则可以为 Microsoft.Storage 创建服务终结点，使所有 Blob 存储流量通过 Azure 主干。 也可以采用此方法，而不是为 Blob 存储创建用户定义的路由。

> [!NOTE]
>
> 如果要从 Azure Databricks（如 CosmosDB、Azure Synapse Analytics 等）访问其他 PaaS Azure 数据服务，还必须将用户定义的路由添加到这些服务的路由表。 使用 `nslookup` 或等效的命令将每个终结点解析到其 IP 地址。

## <a name="step-4-validate-the-setup"></a><a id="on-prem-test"> </a><a id="step-4-validate-the-setup"> </a>步骤 4：验证设置

要验证设置：

1. 在 Azure Databricks 工作区中创建群集。

   如果此操作失败，请再次执行步骤 1-3 中的说明，尝试注释中提到的备用配置。

   如果仍然无法创建群集，请检查路由表是否具有所有所需的用户定义的路由。 如果对 Blob 存储使用服务终结点而不是用户定义的路由，也请检查该配置。

   如果失败，请联系你的 Microsoft 和 Databricks 帐户团队寻求帮助。

2. 创建群集后，尝试使用 `%sh` 从笔记本进行简单 ping 来连接到本地 VM。

在进行故障排除时，以下指南也很有帮助：

* [故障排除](vnet-inject.md#vnet-inject-troubleshoot)
* [解决问题](/virtual-network/diagnose-network-routing-problem#resolve-a-problem)

## <a name="option-route-azure-databricks-traffic-using-a-virtual-appliance-or-firewall"></a><a id="option-route-azure-databricks-traffic-using-a-virtual-appliance-or-firewall"> </a><a id="route-via-firewall"> </a>选项：使用虚拟设备或防火墙路由 Azure Databricks 流量

你可能还需要使用防火墙或 DLP 设备（如 Azure 防火墙、Palo Alto、Barracuda 等）筛选或审核 Azure Databricks 群集节点的所有传出流量。 这可能需要执行以下操作：

* 满足要求“检查”并允许或拒绝配置的所有传出流量的企业安全策略。
* 获取所有 Azure Databricks 群集的一个类 NAT 公共 IP 或 CIDR，其可在任何数据源的允许列表中进行配置。

若要设置此方式的筛选，请按照步骤 1-4 以及一些附加步骤中的说明操作。 以下内容仅供参考；详细信息可能因防火墙设备而异：

1. 按照[创建 NVA](/virtual-network/tutorial-create-route-table-portal#create-an-nva) 中的说明在传输 VNet 中设置虚拟设备或防火墙。

   或者，可以在 Azure Databricks VNet 中的安全或 DMZ 子网中创建防火墙，该子网与现有专用子网和公共子网分开。 但是，如果需要为多个工作区设置防火墙，则建议将传输 VNet 解决方案作为中心辐射型拓扑。

2. 在[自定义路由表](#create-routes)中创建到 0.0.0.0/0 的附加路由，并有类型为“虚拟设备”的下一跃点和适当的下一跃点地址。

   在步骤 3 中配置的路由应保留，但如果需要通过防火墙路由 Blob 存储流量，则可以删除 Blob 存储的路由（或服务终结点）。

   如果使用安全或 DMZ 子网方法，则可能需要创建一个附加路由表以仅与该子网关联。 该路由表应具有到 0.0.0.0 的路由，且下一跃点类型为“Internet”或“虚拟网络网关”，具体取决于流量是直接发往公共网络还是通过本地网络。

3. 在防火墙设备中配置“允许”和“拒绝”规则。

   如果已删除 Blob 存储的路由，则应将这些路由添加到防火墙的允许列表中。

   可能还需要将某些公共存储库（如 Ubuntu 等）添加到允许列表，以确保正确创建群集。

   有关允许列表的信息，请参阅[用户定义的 Azure Databricks 路由设置](udr.md)。

## <a name="option-configure-custom-dns"></a><a id="option-configure-custom-dns"> </a><a id="vnet-custom-dns"> </a>选项：配置自定义 DNS

如果要使用自己的 DNS 进行名称解析，可以使用[部署在你自己的虚拟网络中](vnet-inject.md)的 Azure Databricks 工作区进行此操作。 有关如何为 Azure 虚拟网络配置自定义 DNS 的详细信息，请参阅以下 Microsoft 文档：

* [使用自己的 DNS 服务器的名称解析](/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#name-resolution-that-uses-your-own-dns-server)
* [指定 DNS 服务器](/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#specify-dns-servers)