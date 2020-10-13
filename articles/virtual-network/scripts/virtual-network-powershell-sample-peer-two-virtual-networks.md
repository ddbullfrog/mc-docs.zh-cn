---
title:将两个虚拟网络对等互连 - Azure PowerShell 脚本示例 description:在同一区域中创建并连接两个虚拟网络。 使用两个对等虚拟网络的 Azure 脚本通过 Azure 网络来连接网络。
services: virtual-network documentationcenter: virtual-network manager: mtillman ms.service: virtual-network ms.devlang: powershell ms.topic: sample ms.tgt_pltfrm: ms.workload： infrastructure origin.date:03/20/2018 author: rockboyfor ms.date:10/05/2020 ms.testscope: yes ms.testdate:08/10/2020 ms.author: v-yeche ms.custom: devx-track-azurepowershell
---

# <a name="peer-two-virtual-networks-script-sample"></a>将两个虚拟网络脚本示例对等互连

此脚本示例通过 Azure 网络在同一区域创建并连接两个虚拟网络。 运行脚本后，会在两个虚拟网络之间创建对等互连。

可以通过本地 PowerShell 安装来执行脚本。 如果在本地使用 PowerShell，则此脚本需要 Az PowerShell 模块 5.4.1 或更高版本。 要查找已安装的版本，请运行 `Get-Module -ListAvailable Az`。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 来创建与 Azure 的连接。

<!-- Not Available on [Cloud Shell](https://shell.azure.com/powershell) -->

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

<!--[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]-->

```powershell
# Variables for common values
$rgName='MyResourceGroup'
$location='chinaeast'

# Create a resource group.
New-AzResourceGroup -Name $rgName -Location $location

# Create virtual network 1.
$vnet1 = New-AzVirtualNetwork -ResourceGroupName $rgName -Name 'Vnet1' -AddressPrefix '10.0.0.0/16' -Location $location

# Create virtual network 2.
$vnet2 = New-AzVirtualNetwork -ResourceGroupName $rgName -Name 'Vnet2' -AddressPrefix '10.1.0.0/16' -Location $location

# Peer VNet1 to VNet2.
Add-AzVirtualNetworkPeering -Name 'LinkVnet1ToVnet2' -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id

# Peer VNet2 to VNet1.
Add-AzVirtualNetworkPeering -Name 'LinkVnet2ToVnet1' -VirtualNetwork $vnet2 -RemoteVirtualNetworkId $vnet1.Id

```

## <a name="clean-up-deployment"></a>清理部署

运行以下命令来删除资源组、VM 和所有相关资源：

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 下表中的每条命令均链接到特定于命令的文档：

| 命令 | 注释 |
|---|---|
| [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork)| 创建 Azure 虚拟网络和子网。 |
| [Add-AzVirtualNetworkPeering](https://docs.microsoft.com/powershell/module/az.network/add-azvirtualnetworkpeering) | 创建两个虚拟网络之间的对等互连。  |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/)。

可在[虚拟网络 PowerShell 示例](../powershell-samples.md)中查找其他虚拟网络 PowerShell 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->