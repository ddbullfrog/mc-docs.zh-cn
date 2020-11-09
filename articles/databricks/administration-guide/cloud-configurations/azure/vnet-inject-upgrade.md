---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/28/2020
title: 将“VNet 注入”预览版工作区升级到正式发布版 - Azure Databricks
description: 如果已经在自己的 Azure 虚拟网络中部署了 Azure Databricks 的预览实例（这一功能有时称为 VNet 注入），则可以使用这些说明升级到正式发布版实例。
ms.openlocfilehash: 29d2b45b95778ce52c28a38c3e707893d7e1b896
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106517"
---
# <a name="upgrade-your-vnet-injection-preview-workspace-to-ga"></a><a id="upgrade-your-vnet-injection-preview-workspace-to-ga"></a><a id="vnet-inject-upgrade"></a>将“VNet 注入”预览版工作区升级到正式发行版

[将 Azure Databricks 工作区部署到自己的 Azure 虚拟网络](vnet-inject.md)（有时称为 VNet 注入）这一功能现已从预览版升级为正式版，因此应在 2020 年 3 月 31 日前将预览工作区升级到正式发布版。 如果不升级会导致工作区功能丢失。 2020 年 6 月 1 日之后，你将无法访问你的工作区。

> [!IMPORTANT]
>
> 如果你在 6 月 1 日之前没有升级工作区，则将无法访问工作区。 6 月 1 日之后，请遵循升级步骤，然后开具支持工单以重新获取对工作区的访问权限。

在 VNet 注入的正式发布版中，与预览版不同，Azure Databricks 管理 Azure Databricks 部署所需的所有网络安全组 (NSG) 规则。 因此，升级过程涉及将公共子网和专用子网委派给 `Microsoft.Databricks/workspaces` 服务，从而允许 Azure Databricks 维护那些网络安全组规则。 此委派不会授予 Azure Databricks 任何权限来更新你可以自行添加到子网的网络安全组规则。

此过程不会干扰现有的 Azure Databricks 群集或正在运行的作业，并且不会对 Azure Databricks 工作区进行任何可见的更改。

## <a name="requirements"></a>要求

必须具有以下权限：`Microsoft.Network/virtualNetworks/subnets/write`。 默认情况下，拥有“所有者”或“参与者”角色的用户具有此权限。 若要了解如何分配此权限，请参阅[权限](/virtual-network/manage-subnet-delegation#permissions)。

## <a name="upgrade-using-azure-cli"></a>使用 Azure CLI 进行升级

1. 登录 Azure CLI。

   ```powershell
   az login
   ```

2. 设置环境变量。

   ```ini
   subscriptionId=<Your Subscription ID>
   vnetName=<Your Virtual Network’s Name>
   rgName=<Your Virtual Network’s Resource Group>
   publicSubnetName=<Name of Your Virtual Network’s Public Subnet>
   privateSubnetName=<Name of Your Virtual Network’s Private Subnet>
   delegation='Microsoft.Databricks/workspaces'
   ```

3. 将公共子网委托给 Azure Databricks。

   ```powershell
   az network vnet subnet update --subscription $subscriptionId --resource-group $rgName --vnet-name $vnetName --name $publicSubnetName --delegation $delegation
   ```

4. 将专用子网委托给 Azure Databricks。

   ```powershell
   az network vnet subnet update --subscription $subscriptionId --resource-group $rgName --vnet-name $vnetName --name $privateSubnetName --delegation $delegation
   ```

## <a name="upgrade-using-powershell"></a>使用 PowerShell 进行升级

1. 安装网络模块。

   ```powershell
   Install-Module -Name Az.Network -AllowClobber -Force
   ```

2. 设置环境变量。

   ```ini
   $subscriptionId = <Your Subscription ID>
   $vnetName = <Your Virtual Network Name>
   $rgname = <Your Virtual Network's Resource Group>
   $delegation = 'Microsoft.Databricks/workspaces'
   $publicSubnetName = <Name of Your Virtual Network’s Public Subnet>
   $privateSubnetName = <Name of Your Virtual Network’s Private Subnet>
   ```

3. 在 shell 中设置订阅。

   ```powershell
   Select-AzSubscription -SubscriptionId $subscriptionId
   ```

4. 检索虚拟网络和相应的子网。

   ```powershell
   $vNet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgname
   $publicSubnet = Get-AzVirtualNetworkSubnetConfig -name $publicSubnetName -VirtualNetwork $vNet
   $privateSubnet = Get-AzVirtualNetworkSubnetConfig -name $privateSubnetName -VirtualNetwork $vNet
   ```

5. 创建到 Azure Databricks 的新委派。

   ```powershell
   $delegation = New-AzDelegation -Name adbDelegation -ServiceName "Microsoft.Databricks/workspaces"
   ```

6. 将公用和专用子网设置为新委派并更新虚拟网络。

   ```powershell
   Set-AzVirtualNetworkSubnetConfig -Name $publicSubnet.Name -VirtualNetwork $vNet -Delegation $delegation -AddressPrefix $publicSubnet.AddressPrefix

   Set-AzVirtualNetworkSubnetConfig -Name $privateSubnet.Name -VirtualNetwork $vNet -Delegation $delegation -AddressPrefix $privateSubnet.AddressPrefix

   Set-AzVirtualNetwork -VirtualNetwork $vNet
   ```

## <a name="upgrade-using-the-azure-portal"></a>使用 Azure 门户进行升级

1. 在 Azure 门户中，导航到部署了 Azure Databricks 工作区的虚拟网络。 请参阅[查看虚拟网络和设置](/virtual-network/manage-virtual-network#view-virtual-networks-and-settings)。

   > [!div class="mx-imgBorder"]
   > ![虚拟网络设置](../../../_static/images/vnet/vnet-inject-vnet.png)

2. 在左侧菜单中，单击“子网”。 你将看到显示的专用和公共子网信息。

   > [!div class="mx-imgBorder"]
   > ![子网](../../../_static/images/vnet/vnet-inject-subnet.png)

3. 单击公共子网行，转到“子网委派”下拉列表并选择“Microsoft.Databricks/workspaces”服务 。

   > [!div class="mx-imgBorder"]
   > ![子网委派](../../../_static/images/vnet/vnet-inject-subnet-delegation.png)

   有关子网委派的详细信息，请参阅[添加或删除子网委派](/virtual-network/manage-subnet-delegation)。

4. 对专用子网重复子网委派。
5. 保存所做更改。

## <a name="upgrade-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板进行升级

> [!IMPORTANT]
>
> 如果在预览期间使用 Azure 资源管理器 (ARM) 模板将 Azure Databricks 工作区部署到自己的虚拟网络，并且想要继续使用 Azure 资源管理器模板来创建虚拟网络和部署工作区，则应使用“升级的 Azure 资源管理器模板”。 请参阅[配置虚拟网络](vnet-inject.md#vnet-inject-advanced)。

## <a name="post-upgrade-steps"></a>升级后的步骤

完成子网委派后，Azure Databricks 将在 24 小时内完成工作区升级。 升级完成后，应会在附加到公共和专用子网的网络安全组中看到[一组新的网络安全规则](vnet-inject.md#nsg)。 其中每个规则名称都以前缀 `Microsoft.Databricks-workspaces` 开头。 任何以前缀 `databricks` 开头的规则不再是必需的，应使用以下过程删除：

1. 在 Azure 门户中，导航到部署了 Azure Databricks 工作区的虚拟网络。 请参阅[查看虚拟网络和设置](/virtual-network/manage-virtual-network#view-virtual-networks-and-settings)。

   > [!div class="mx-imgBorder"]
   > ![虚拟网络设置](../../../_static/images/vnet/vnet-inject-vnet.png)

2. 在左侧菜单中，单击“子网”，然后复制专用子网和公共子网的网络安全组的名称。

   > [!div class="mx-imgBorder"]
   > ![子网](../../../_static/images/vnet/vnet-inject-find-nsg.png)

3. 将公共子网的网络安全组名称粘贴到搜索栏中，以打开“网络安全组概述”页。
4. 在“概述”页上，找到所有以“databricks”开头的入站和出站规则并删除它们。

   > [!div class="mx-imgBorder"]
   > ![网络安全组概述](../../../_static/images/vnet/vnet-inject-delete-rules.png)

5. 对专用子网重复前两个步骤。