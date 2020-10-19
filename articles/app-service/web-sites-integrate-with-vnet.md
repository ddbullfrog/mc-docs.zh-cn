---
title: 将应用与 Azure 虚拟网络集成
description: 将 Azure 应用服务中的应用与 Azure 虚拟网络集成。
author: ccompy
ms.assetid: 90bc6ec6-133d-4d87-a867-fcf77da75f5a
ms.topic: article
origin.date: 08/05/2020
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: seodec18
ms.openlocfilehash: 72fc9a0e55ec07ab4bb36f8849ae6e536a385150
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170719"
---
# <a name="integrate-your-app-with-an-azure-virtual-network"></a>将应用与 Azure 虚拟网络集成

本文介绍 Azure 应用服务 VNet 集成功能，并介绍如何为 [Azure 应用服务](overview.md)中的应用设置此功能。 使用 [Azure 虚拟网络][VNETOverview] (VNet) 可将多个 Azure 资源置于无法通过 Internet 路由的网络中。

Azure 应用服务有两种变体：

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-types.md)]

## <a name="enable-vnet-integration"></a>启用 VNet 集成

1. 在应用服务门户中转到“网络”UI。 在“VNet 集成”下，选择“单击此处进行配置”。 

1. 选择“添加 VNet”。

   ![选择 VNet 集成][1]

1. 下拉列表包含订阅内位于相同区域中的所有 Azure 资源管理器虚拟网络。 下面是所有其他区域中资源管理器虚拟网络的列表。 选择要集成的 VNet。

   ![选择 VNet][2]

   <!-- * If the VNet is in the same region, either create a new subnet or select an empty preexisting subnet.  -->

   * 若要选择另一区域中的 VNet，必须已预配一个启用了点到站点连接的 VNet 网关。
   * 若要与经典 VNet 集成，请不要选择“虚拟网络”下拉列表，而应选择“单击此处连接到经典 VNet”。  选择所需的经典虚拟网络。 目标 VNet 中必须已预配一个启用了点到站点连接的虚拟网关。

    ![选择经典 VNet][3]

在集成期间，应用会重启。 完成集成后，将会看到与之集成的 VNet 的详细信息。
<!-- ## Regional VNet Integration -->
<!-- Add VNet (preview) not available in mc portal-->

## <a name="gateway-required-vnet-integration"></a>需要网关的 VNet 集成

需要网关的 VNet 集成支持连接到另一区域中的 VNet，或连接到经典虚拟网络。 需要网关的 VNet 集成：

* 允许应用一次只连接到一个 VNet。
* 允许在一个应用服务计划中最多集成 5 个 VNet。
* 允许在应用服务计划中由多个应用使用同一个 VNet，不影响可供应用服务计划使用的总数。 如果有 6 个应用在使用同一应用服务计划中的同一 VNet，则算作是使用了一个 VNet。
* 由于 SLA 是基于网关，因此可实现 99.9% 的 SLA。
* 允许应用使用配置给 VNet 的 DNS。
* 需要在基于虚拟网络路由的网关中配置 SSTP 点到站点 VPN，然后才能将其连接到应用。

需要网关的 VNet 集成不可用于：

* 通过 Azure ExpressRoute 连接的 VNet。
* 从 Linux 应用
* 访问服务终结点保护的资源。
* 既支持 ExpressRoute，也支持点到站点 VPN 或站点到站点 VPN 的共存网关。

### <a name="set-up-a-gateway-in-your-azure-virtual-network"></a>在 Azure 虚拟网络中设置网关 ###

若要创建网关，请执行以下操作：

1. 在 VNet 中[创建网关子网][creategatewaysubnet]。

1. [创建 VPN 网关][creategateway]。 选择基于路由的 VPN 类型。

1. [设置点到站点地址][setp2saddresses]。 如果网关不在基本 SKU 中，则必须在点到站点配置中禁用 IKEV2 并选择 SSTP。 点到站点地址空间必须在 RFC 1918 地址块 10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16 中。

如果创建用于应用服务 VNet 集成的网关，则不需要上传证书。 创建网关可能需要 30 分钟。 若要将应用与 VNet 集成，必须先预配网关。

### <a name="how-gateway-required-vnet-integration-works"></a>需要网关的 VNet 集成的工作原理

需要网关的 VNet 集成基于点到站点 VPN 技术。 点到站点 VPN 将网络访问限制于可托管应用的虚拟机。 应用受到限制，只能通过混合连接或 VNet 集成向外发送流量至 Internet。 通过门户将应用配置为使用需要网关的 VNet 集成后，系统会代你管理复杂的协商，以便在网关上和应用程序端创建并分配证书。 结果是，用于托管应用的辅助角色能够直接连接到所选 VNet 中的虚拟网关。

![需要网关的 VNet 集成的工作原理][6]

### <a name="access-on-premises-resources"></a>访问本地资源

应用可以通过与具备站点到站点连接的 VNet 集成来访问本地资源。 如果使用需要网关的 VNet 集成，请使用点到站点地址块更新本地 VPN 网关路由。 先设置站点到站点 VPN，接着应通过用于配置该 VPN 的脚本来正确地设置路由。 如果在创建站点到站点地址后才添加点到站点 VPN，则需手动更新路由。 操作详情取决于每个网关，在此不作说明。 不能使用站点到站点 VPN 连接来配置 BGP。

<!-- There is no additional configuration required for the regional VNet Integration feature to reach through your VNet, and  -->

本地连接。 只需使用 ExpressRoute 或站点到站点 VPN 将 VNet 连接到本地。 

> [!NOTE]
> 需要网关的 VNet 集成功能不将应用与包含 ExpressRoute 网关的 VNet 集成。 即使以[共存模式][VPNERCoex]配置 ExpressRoute 网关，VNet 集成也不会生效。
> 

<!--  If you need to access resources through an ExpressRoute connection, then you can use the regional VNet Integration feature or an [App Service Environment][ASE], which runs in your VNet.  -->

### <a name="peering"></a>对等互连

<!-- If you use peering with the regional VNet Integration, you don't need to do any additional configuration.  -->

如果结合对等互连使用需要网关的 VNet 集成，则需要配置几个附加的项。 若要配置对等互连以使用应用，请执行以下操作：

1. 在应用所连接的 VNet 上添加对等互连连接。 在添加对等互连连接时，请启用“允许虚拟网络访问”并选择“允许转发流量”和“允许网关传输”  。
1. 在与所连接的 VNet 对等互连的 VNet 上添加对等互连连接。 在目标 VNet 上添加对等互连连接时，请启用“允许虚拟网络访问”并选择“允许转发流量”和“允许远程网关”  。
1. 在门户中转到“应用服务计划” > “网络” > “VNet 集成”UI。 选择应用连接的 VNet。 在路由部分，添加与应用所连接的 VNet 对等互连的 VNet 的地址范围。

## <a name="manage-vnet-integration"></a>管理 VNet 集成

与 VNet 连接和断开连接都在应用级别进行的。 可能影响多个应用的 VNet 集成的操作在应用服务计划级别执行。 可以通过应用 >“网络” > “VNet 集成”门户获取 VNet 的详细信息。 可以在“应用服务计划” > “网络” > “VNet 集成”门户中查看应用服务计划级别的类似信息。

在 VNet 集成实例的应用视图中，能够执行的唯一操作是断开应用与当前连接到的 VNet 的连接。 若要断开应用与 VNet 的连接，请选择“断开连接”。 断开与 VNet 的连接后，应用会重启。 断开连接操作不会更改 VNet。 不会删除子网或网关。 若要删除 VNet，请先断开应用与该 VNet 的连接，然后删除该 VNet 中的资源，例如网关。

应用服务计划 VNet 集成 UI 会显示应用服务计划中的应用使用的所有 VNet 集成。 若要查看单个 VNet 的详细信息，请选择你感兴趣的 VNet。 在此处，可以针对需要网关的 VNet 集成执行两项操作：

* **同步网络**：同步网络操作仅用于网关相关的 VNet 集成功能。 执行同步网络操作确保了证书与网络信息是同步的。如果添加或更改 VNet 的 DNS，请执行同步网络操作。 此操作重启使用此 VNet 的任何应用。 如果你使用的是属于不同订阅的应用和 VNet，此操作无效。
* **添加路由**：添加路由会促使出站流量进入 VNet。

### <a name="gateway-required-vnet-integration-routing"></a>需要网关的 VNet 集成路由
在 VNet 中定义的路由用于将流量从应用导入 VNet。 若要将其他出站流量发送到 VNet 中，请在此处添加相关地址块。 此功能只适用于网关所需的 VNet 集成。 使用需要网关的 VNet 集成时，路由表不会影响应用流量。

### <a name="gateway-required-vnet-integration-certificates"></a>需要网关的 VNet 集成证书
启用需要网关的 VNet 集成后，必须进行证书交换以确保连接的安全性。 除了证书，还有 DNS 配置、路由以及其他类似的用于描述网络的内容。

如果更改了证书或网络信息，请选择“同步网络”。 选择“同步网络”会导致应用与 VNet 之间的连接出现短暂的中断。 虽然应用不会重启，但失去连接会导致站点功能失常。

## <a name="pricing-details"></a>定价详细信息
<!-- The regional VNet Integration feature has no additional charge for use beyond the App Service plan pricing tier charges. -->

使用需要网关的 VNet 集成功能会产生三项相关费用：

* **应用服务计划定价层费用**：应用必须属于“标准”、“高级”或“高级 V2”应用服务计划。 有关这些费用的详细信息，请参阅[应用服务定价][ASPricing]。
* **数据传输费用**：传出数据会产生费用，即使 VNet 位于同一数据中心也是如此。 [数据传输定价详细信息][DataPricing]中对这些费用进行了说明。
* **VPN 网关费用**：点到站点 VPN 所需的虚拟网关会产生费用。 有关详细信息，请参阅 [VPN 网关定价][VNETPricing]。

## <a name="troubleshooting"></a>疑难解答

[!INCLUDE [app-service-web-vnet-troubleshooting](../../includes/app-service-web-vnet-troubleshooting.md)]

## <a name="automation"></a>自动化

<!-- regional VNet Integration
CLI support is available for regional VNet Integration. To access the following commands, [install the Azure CLI][installCLI].

```azurecli
az webapp vnet-integration --help

Group
    az webapp vnet-integration : Methods that list, add, and remove virtual network
    integrations from a webapp.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    add    : Add a regional virtual network integration to a webapp.
    list   : List the virtual network integrations on a webapp.
    remove : Remove a regional virtual network integration from webapp.

az appservice vnet-integration --help

Group
    az appservice vnet-integration : A method that lists the virtual network
    integrations used in an appservice plan.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    list : List the virtual network integrations used in an appservice plan.
```

PowerShell support for regional VNet integration is available too, but you must create generic resource with a property array of the subnet resourceID

```azurepowershell
# Parameters
$sitename = 'myWebApp'
$resourcegroupname = 'myRG'
$VNetname = 'myVNet'
$location = 'myRegion'
$integrationsubnetname = 'myIntegrationSubnet'
$subscriptionID = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee'

#Property array with the SubnetID
$properties = @{
  subnetResourceId = "/subscriptions/$subscriptionID/resourceGroups/$resourcegroupname/providers/Microsoft.Network/virtualNetworks/$VNetname/subnets/$integrationsubnetname"
}

#Creation of the VNet integration
$vNetParams = @{
  ResourceName = "$sitename/VirtualNetwork"
  Location = $location
  ResourceGroupName = $resourcegroupname
  ResourceType = 'Microsoft.Web/sites/networkConfig'
  PropertyObject = $properties
}
New-AzResource @vNetParams
```


For gateway-required VNet Integration, you can integrate App Service with an Azure virtual network by using PowerShell. For a ready-to-run script, see [Connect an app in Azure App Service to an Azure virtual network](https://gallery.technet.microsoft.com/scriptcenter/Connect-an-app-in-Azure-ab7527e3).


<!--Image references-->
[1]: ./media/web-sites-integrate-with-vnet/vnetint-app.png
[2]: ./media/web-sites-integrate-with-vnet/vnetint-addvnet.png
[3]: ./media/web-sites-integrate-with-vnet/vnetint-classic.png
[5]: ./media/web-sites-integrate-with-vnet/vnetint-regionalworks.png
[6]: ./media/web-sites-integrate-with-vnet/vnetint-gwworks.png


<!--Links-->
[VNETOverview]: ../virtual-network/virtual-networks-overview.md
[AzurePortal]: https://portal.azure.cn/
[ASPricing]: https://www.azure.cn/pricing/details/app-service/
[VNETPricing]: https://www.azure.cn/pricing/details/vpn-gateway/
[DataPricing]: https://www.azure.cn/pricing/details/data-transfer/
[V2VNETP2S]: ../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md
[ILBASE]: environment/create-ilb-ase.md
[V2VNETPortal]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md
[VPNERCoex]: ../expressroute/expressroute-howto-coexist-resource-manager.md
[ASE]: environment/intro.md
[creategatewaysubnet]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw
[creategateway]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw
[setp2saddresses]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#addresspool
[VNETRouteTables]: ../virtual-network/manage-route-table.md
[installCLI]: https://docs.azure.cn/cli/install-azure-cli?view=azure-cli-latest/
