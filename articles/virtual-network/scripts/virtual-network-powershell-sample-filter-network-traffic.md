---
title: 筛选 VM 网络流量 - Azure PowerShell 脚本示例
description: 筛选入站和出站 VM 网络流量 - Azure PowerShell 脚本示例。
services: virtual-network
documentationcenter: virtual-network
manager: twooley
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
origin.date: 03/20/2018
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: yes
ms.testdate: 08/10/2020
ms.author: v-yeche
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f211f433f79da1f87409ade89075b1f975cc32ac
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564446"
---
# <a name="filter-inbound-and-outbound-vm-network-traffic-script-sample"></a>筛选入站和出站 VM 网络流量脚本示例

该脚本示例创建了包含前端和后端子网的虚拟网络。 前端子网的入站网络流量仅限于 HTTP 和 HTTPS，不允许从后端子网到 Internet 的出站流量。 运行该脚本后，将具有一个包含两个 NIC 的虚拟机。 每个 NIC 连接到不同的子网。

可以通过本地 PowerShell 安装来执行脚本。 如果在本地使用 PowerShell，则此脚本需要 Azure PowerShell 模块 1.0.0 版或更高版本。 要查找已安装的版本，请运行 `Get-Module -ListAvailable Az`。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 来创建与 Azure 的连接。

<!-- Not Available on [Cloud Shell](https://shell.azure.com/bash) -->

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

<!--[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]-->

```powershell
# Variables for common values
$rgName='MyResourceGroup'
$location='chinaeast'

# Create user object
$cred = Get-Credential -Message 'Enter a username and password for the virtual machine.'

# Create a resource group.
New-AzResourceGroup -Name $rgName -Location $location

# Create a virtual network, a front-end subnet, and a back-end subnet.
$fesubnet = New-AzVirtualNetworkSubnetConfig -Name 'MySubnet-FrontEnd' -AddressPrefix '10.0.1.0/24'
$besubnet = New-AzVirtualNetworkSubnetConfig -Name 'MySubnet-BackEnd' -AddressPrefix '10.0.2.0/24'

$vnet = New-AzVirtualNetwork -ResourceGroupName $rgName -Name 'MyVnet' -AddressPrefix '10.0.0.0/16' `
  -Location $location -Subnet $fesubnet, $besubnet

# Create NSG rules to allow HTTP & HTTPS traffic inbound.
$rule1 = New-AzNetworkSecurityRuleConfig -Name 'Allow-HTTP-ALL' -Description 'Allow HTTP' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 80

$rule2 = New-AzNetworkSecurityRuleConfig -Name 'Allow-HTTPS-All' -Description 'Allow HTTPS' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 200 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 443

# Create an NSG rule to allow RDP traffic in from the Internet to the front-end subnet.
$rule3 = New-AzNetworkSecurityRuleConfig -Name 'Allow-RDP-All' -Description 'Allow RDP' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 300 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 3389

# Create a network security group (NSG) for the front-end subnet.
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RgName -Location $location `
  -Name "MyNsg-FrontEnd" -SecurityRules $rule1,$rule2,$rule3

# Associate the front-end NSG to the front-end subnet.
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name 'MySubnet-FrontEnd' `
  -AddressPrefix 10.0.1.0/24 -NetworkSecurityGroup $nsg

# Create an NSG rule to block all outbound traffic from the back-end subnet to the Internet (inbound blocked by default).
$rule1 = New-AzNetworkSecurityRuleConfig -Name 'Deny-Internet-All' -Description "Deny all Internet" `
  -Access Allow -Protocol Tcp -Direction Outbound -Priority 100 `
  -SourceAddressPrefix * -SourcePortRange * `
  -DestinationAddressPrefix Internet -DestinationPortRange *

# Create a network security group for the back-end subnet.
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RgName -Location $location `
  -Name "MyNsg-BackEnd" -SecurityRules $rule1

# Associate the back-end NSG to the back-end subnet.
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name 'MySubnet-backEnd' `
  -AddressPrefix 10.0.2.0/24 -NetworkSecurityGroup $nsg

# Create a public IP address for the VM front-end network interface.
$publicipvm = New-AzPublicIpAddress -ResourceGroupName $rgName -Name 'MyPublicIp-FrontEnd' `
  -location $location -AllocationMethod Dynamic

# Create a network interface for the VM attached to the front-end subnet.
$nicVMfe = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name MyNic-FrontEnd -PublicIpAddress $publicipvm -Subnet $vnet.Subnets[0]

# Create a network interface for the VM attached to the back-end subnet.
$nicVMbe = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name MyNic-BackEnd -Subnet $vnet.Subnets[1]

# Create the VM with both the FrontEnd and BackEnd NICs.
$vmConfig = New-AzVMConfig -VMName 'myVM' -VMSize Standard_DS2 | `
  Set-AzVMOperatingSystem -Windows -ComputerName 'myVM' -Credential $cred | `
  Set-AzVMSourceImage -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' `
  -Skus '2016-Datacenter' -Version 'latest'

$vmconfig = Add-AzVMNetworkInterface -VM $vmConfig -id $nicVMfe.Id -Primary
$vmconfig = Add-AzVMNetworkInterface -VM $vmConfig -id $nicVMbe.Id

# Create a virtual machine
$vm = New-AzVM -ResourceGroupName $rgName -Location $location -VM $vmConfig

```

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源：

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
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

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/)。

可在[虚拟网络 PowerShell 示例](../powershell-samples.md)中查找其他虚拟网络 PowerShell 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->