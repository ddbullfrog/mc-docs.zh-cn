---
title: 创建 Azure 防火墙测试环境
description: 此脚本示例创建防火墙和测试网络环境。 网络具有一个包含三个子网的 VNet。
services: virtual-network
ms.service: firewall
ms.devlang: powershell
ms.topic: sample
origin.date: 11/19/2019
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.openlocfilehash: a41a9fe43a178808f6b0fea9ffa347c1f3e80786
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246793"
---
# <a name="create-an-azure-firewall-test-environment"></a>创建 Azure 防火墙测试环境

此脚本示例创建防火墙和测试网络环境。 网络有一个 VNet，其中包含三个子网：*AzureFirewallSubnet*、*ServersSubnet* 和 *JumpboxSubnet*。 ServersSubnet 和 JumpboxSubnet 每个中都有一个 2 核 Windows Server。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

防火墙在 AzureFirewallSubnet 中并配置有一个应用程序规则集合，其中包含允许访问 `www.microsoft.com` 的单个规则。

创建了用户定义的一个路由，它引导来自 ServersSubnet 的网络流量穿过应用了防火墙规则的防火墙。

可以通过本地 PowerShell 安装来运行脚本。 

<!--Not Available on Azure Cloud Shell-->

如果在本地运行 PowerShell，则此脚本需要 Azure PowerShell。 要查找已安装的版本，请运行 `Get-Module -ListAvailable Az`。 

如果需要升级，则可以使用 `PowerShellGet`，它内置在 Windows 10 和 Windows Server 2016 中。

> [!NOTE]
>对于其他 Windows 版本，需要先安装 `PowerShellGet`，然后才能使用它。 可以运行 `Get-Module -Name PowerShellGet -ListAvailable | Select-Object -Property Name,Version,Path` 来确定它是否已安装在你的系统上。 如果输出为空，则需要安装最新的 [Windows Management Framework](https://www.microsoft.com/download/details.aspx?id=54616)。

有关详细信息，请参阅[安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps)

使用 Web 平台安装程序执行的任何现有 Azure PowerShell 安装都将与 PowerShellGet 安装冲突并且需要删除。

请注意，如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 来创建与 Azure 的连接。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

```powershell

#ResourceGroup name and location
$RG="AzfwSampleScriptChinaEast"
$Location="China East"

#User credentials for JumpBox and Server VMs
$securePassword = ConvertTo-SecureString 'P@$$W0rd010203' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("AzfwUser", $securePassword)

#Create new RG
New-AzResourceGroup -Name $RG -Location $Location

#Create Vnet
$VnetName=$RG+"Vnet"
New-AzVirtualNetwork -ResourceGroupName $RG -Name $VnetName -AddressPrefix 192.168.0.0/16 -Location $Location

#Configure subnets
$vnet = Get-AzVirtualNetwork -ResourceGroupName $RG -Name $VnetName
Add-AzVirtualNetworkSubnetConfig -Name AzureFirewallSubnet -VirtualNetwork $vnet -AddressPrefix 192.168.1.0/24
Add-AzVirtualNetworkSubnetConfig -Name JumpBoxSubnet -VirtualNetwork $vnet -AddressPrefix 192.168.0.0/24
Add-AzVirtualNetworkSubnetConfig -Name ServersSubnet -VirtualNetwork $vnet -AddressPrefix 192.168.2.0/24
Set-AzVirtualNetwork -VirtualNetwork $vnet

#create Public IP for jumpbox and LB
$LBPipName = $RG + "PublicIP"
$LBPip = New-AzPublicIpAddress -Name $LBPipName  -ResourceGroupName $RG -Location $Location -AllocationMethod Static -Sku Standard
$JumpBoxpip = New-AzPublicIpAddress -Name "JumpHostPublicIP"  -ResourceGroupName $RG -Location $Location -AllocationMethod Static -Sku Basic

# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleSSH  -Protocol Tcp -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow

# Create a network security group
$NsgName = $RG+"NSG"
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RG -Location $Location -Name $NsgName -SecurityRules $nsgRuleRDP

#Create jumpbox
$vnet = Get-AzVirtualNetwork -ResourceGroupName $RG -Name $VnetName
$JumpBoxSubnetId = $vnet.Subnets[1].Id
# Create a virtual network card and associate with jumpbox public IP address
$JumpBoxNic = New-AzNetworkInterface -Name JumpBoxNic -ResourceGroupName $RG -Location $Location -SubnetId $JumpBoxSubnetId -PublicIpAddressId $JumpBoxpip.Id -NetworkSecurityGroupId $nsg.Id
$JumpBoxConfig = New-AzVMConfig -VMName JumpBox -VMSize Standard_DS1_v2 | Set-AzVMOperatingSystem -Windows -ComputerName JumpBox -Credential $cred | Set-AzVMSourceImage -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version latest | Add-AzVMNetworkInterface -Id $JumpBoxNic.Id
New-AzVM -ResourceGroupName $RG -Location $Location -VM $JumpBoxConfig

#Create Server VM
$ServersSubnetId = $vnet.Subnets[2].Id
$ServerVmNic = New-AzNetworkInterface -Name ServerVmNic -ResourceGroupName $RG -Location $Location -SubnetId $ServersSubnetId
$ServerVmConfig = New-AzVMConfig -VMName ServerVm -VMSize Standard_DS1_v2 | Set-AzVMOperatingSystem -Windows -ComputerName ServerVm -Credential $cred | Set-AzVMSourceImage -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version latest | Add-AzVMNetworkInterface -Id $ServerVmNic.Id
New-AzVM -ResourceGroupName $RG -Location $Location -VM $ServerVmConfig

#Create AZFW
$GatewayName = $RG + "Azfw"
$Azfw = New-AzFirewall -Name $GatewayName -ResourceGroupName $RG -Location $Location -VirtualNetworkName $vnet.Name -PublicIpName $LBPip.Name

#Add a rule to allow *microsoft.com
$Azfw = Get-AzFirewall -ResourceGroupName $RG
$Rule = New-AzFirewallApplicationRule -Name R1 -Protocol "http:80","https:443" -TargetFqdn "*microsoft.com"
$RuleCollection = New-AzFirewallApplicationRuleCollection -Name RC1 -Priority 100 -Rule $Rule -ActionType "Allow"
$Azfw.ApplicationRuleCollections = $RuleCollection
Set-AzFirewall -AzureFirewall $Azfw

#Create UDR rule
$Azfw = Get-AzFirewall -ResourceGroupName $RG
$AzfwRouteName = $RG + "AzfwRoute"
$AzfwRouteTableName = $RG + "AzfwRouteTable"
$IlbCA = $Azfw.IpConfigurations[0].PrivateIPAddress
$AzfwRoute = New-AzRouteConfig -Name $AzfwRouteName -AddressPrefix 0.0.0.0/0 -NextHopType VirtualAppliance -NextHopIpAddress $IlbCA
$AzfwRouteTable = New-AzRouteTable -Name $AzfwRouteTableName -ResourceGroupName $RG -location $Location -Route $AzfwRoute

#associate to Servers Subnet
$vnet.Subnets[2].RouteTable = $AzfwRouteTable
Set-AzVirtualNetwork -VirtualNetwork $vnet

```

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源：

```powershell
Remove-AzResourceGroup -Name AzfwSampleScriptChinaEast -Force
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟网络和网络安全组。 下表中的每条命令均链接到特定于命令的文档：

| Command | 说明 |
|---|---|
| [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | 创建子网配置对象 |
| [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) | 创建 Azure 虚拟网络和前端子网。 |
| [New-AzNetworkSecurityRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecurityruleconfig) | 创建要分配到网络安全组的安全规则。 |
| [New-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecuritygroup) |创建 NSG 规则，允许或阻止特定子网的特定端口。 |
| [Set-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | 将 NSG 关联到子网。 |
| [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) | 创建用于从 Internet 访问 VM 的公共 IP 地址。 |
| [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface) | 创建虚拟网络接口，并将其附加到虚拟网络的前端和后端子网。 |
| [New-AzVMConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azvmconfig) | 创建 VM 配置。 此配置包括 VM 名称、操作系统和管理凭据等信息。 在创建 VM 期间将使用此配置。 |
| [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) | 创建虚拟机。 |
|[Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组及其中包含的所有资源。 |
|[New-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/new-azfirewall)| 创建新的 Azure 防火墙。|
|[Get-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/get-azfirewall)|获取 Azure 防火墙对象。|
|[New-AzFirewallApplicationRule](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallapplicationrule)|创建新的 Azure 防火墙应用程序规则。|
|[Set-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/set-azfirewall)|将更改提交到 Azure 防火墙对象。|

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/)。

<!-- Update_Description: update meta properties, wording update, update link -->