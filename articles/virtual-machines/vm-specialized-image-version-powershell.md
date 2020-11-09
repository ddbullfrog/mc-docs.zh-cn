---
title: 从专用化映像创建 VM
description: 使用共享映像库中的专用化映像创建 VM。
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
origin.date: 05/04/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.reviewer: akjosh
ms.openlocfilehash: df06eaf5b03a51e12c021fcb2a29da85a937843b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106127"
---
<!--Verified successfully-->
# <a name="create-a-vm-using-a-specialized-image"></a>使用专用化映像创建 VM 

从共享映像库中存储的专用化映像版本创建 VM。 要使用通用化映像版本创建 VM，请参阅[使用通用化映像创建 VM](vm-generalized-image-version-powershell.md)。

获得专用化映像版本后，可以使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) cmdlet 创建一个或多个新 VM。 

在此示例中，我们使用映像定义 ID 来确保新 VM 会使用最新版本的映像。 也可通过将映像版本 ID 用作 `Set-AzVMSourceImage -Id` 来使用特定版本。 例如，若要使用映像版本 *1.0.0* ，请键入：`Set-AzVMSourceImage -Id "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"`。 

请注意，使用特定映像版本意味着：如果该特定映像版本由于已删除或已从区域中删除而无法使用，则自动化可能会失败。 建议使用映像定义 ID 来创建新的 VM（除非需要特定的映像版本）。

在此示例中，请根据需要替换资源名称。 

```powershell

# Create some variables for the new VM.

$resourceGroup = "mySIGSpecializedRG"
$location = "China East"
$vmName = "mySpecializedVM"

# Get the image. Replace the name of your resource group, gallery, and image definition. This will create the VM from the latest image version available.

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.

$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet `
   -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name MYvNET `
   -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -Name "mypublicdns$(Get-Random)" `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig `
   -Name myNetworkSecurityGroupRuleRDP  `
   -Protocol Tcp `
   -Direction Inbound `
   -Priority 1000 `
   -SourceAddressPrefix * `
   -SourcePortRange * `
   -DestinationAddressPrefix * `
   -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name myNetworkSecurityGroup `
   -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface `
   -Name $vmName `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using Set-AzVMSourceImage -Id $imageDefinition.Id to use the latest available image version.

$vmConfig = New-AzVMConfig `
   -VMName $vmName `
   -VMSize Standard_D1_v2 | `
   Set-AzVMSourceImage -Id $imageDefinition.Id | `
   Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -VM $vmConfig

```

## <a name="attach-the-data-disk"></a>附加数据磁盘
如果映像包含数据磁盘，则需要将数据磁盘连接到 VM。

```powershell

$vm = Get-AzVM -Name $vmName -ResourceGroupName $resourceGroup 

$lun = $imageVersion.StorageProfile.DataDiskImages.Lun

Add-AzVMDataDisk `
   -CreateOption FromImage `
   -SourceImageUri $imageversion.Id `
   -Lun $lun `
   -Caching $imageVersion.StorageProfile.DataDiskImages.HostCaching `
   -DiskSizeInGB $imageVersion.StorageProfile.DataDiskImages.SizeInGB `
   -VM $vm

```

## <a name="next-steps"></a>后续步骤

<!--Not Available on [Azure Image Builder (preview)](./linux/image-builder-overview.md)-->
<!--Not Available on [create a new image version from an existing image version](./linux/image-builder-gallery-update-image-version.md)-->

此外可以使用模板创建共享映像库资源。 提供多个 Azure 快速入门模板： 

- [创建共享映像库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sig-create/)
- [在共享的映像库中创建映像定义](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sig-image-definition-create/)
- [在共享映像库中创建映像版本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sig-image-version-create/)
- [根据映像版本创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-sig/)

有关共享映像库的详细信息，请参阅[概述](./windows/shared-image-galleries.md)。 如果遇到问题，请参阅[排查共享映像库问题](troubleshooting-shared-images.md)。

<!-- Update_Description: update meta properties, wording update, update link -->