---
title: 从映像版本创建托管磁盘
description: 从共享映像库中的映像版本创建托管磁盘。
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
origin.date: 10/06/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.reviewer: olayemio
ms.openlocfilehash: 52bd1ce6664d90fb0cd4cf38845519897b5c8709
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106663"
---
<!--Verified Successfully-->
# <a name="create-a-managed-disk-from-an-image-version"></a>从映像版本创建托管磁盘

如果需要，可以从存储在共享映像库中的映像版本创建托管磁盘。

## <a name="cli"></a>CLI

将 `source` 变量设置为映像版本的 ID，然后使用 [az disk create](https://docs.azure.cn/cli/disk.md#az_disk_create) 创建托管磁盘。 

可以使用 [az sig image-version list](https://docs.microsoft.com/cli/azure/sig/image-version#az_sig_image_version_list) 查看映像版本列表。 在此示例中，我们将在 myGallery 映像库中查找 myImageDefinition 映像定义中包含的所有映像版本。

<!--CORRECT ON https://docs.microsoft.com/cli/azure/sig/image-version#az_sig_image_version_list-->

```azurecli
az sig image-version list \
   --resource-group myResourceGroup\
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   -o table
```

在此示例中，我们将在 ChinaEast 区域的名为“myResourceGroup”的资源组中创建一个名为“myManagedDisk”的托管磁盘。 

```azurecli
source="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<galleryImageDefinition>/versions/<imageVersion>"

az disk create --resource-group myResourceGroup --location ChinaEast --name myManagedDisk --gallery-image-reference $source 
```

如果该磁盘是数据磁盘，请添加 `--gallery-image-reference-lun` 以指定 LUN。

## <a name="powershell"></a>PowerShell

可以使用 [Get-AzResource](https://docs.microsoft.com/powershell/module/az.resources/get-azresource) 列出所有映像版本。 

```powershell
Get-AzResource `
   -ResourceType Microsoft.Compute/galleries/images/versions | `
   Format-Table -Property Name,ResourceId,ResourceGroupName
```

获得全部所需信息后，可以使用 [Get-AzGalleryImageVersion](https://docs.microsoft.com/powershell/module/az.compute/get-azgalleryimageversion) 来获取要使用的源映像版本并将其分配给变量。 在本示例中，我们将在 `myResourceGroup` 资源组的 `myGallery` 源库中，获取 `myImageDefinition` 定义的 `1.0.0` 映像版本。

```powershell
$sourceImgVer = Get-AzGalleryImageVersion `
   -GalleryImageDefinitionName myImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name 1.0.0
```

为磁盘信息设置一些变量。

```powershell
$location = "China East"
$resourceGroup = "myResourceGroup"
$diskName = "myDisk"
```

使用源映像版本 ID 创建磁盘配置和磁盘。 对于 `-GalleryImageReference`，仅当源为数据磁盘时，才需要 LUN。

```powershell
$diskConfig = New-AzDiskConfig `
   -Location $location `
   -CreateOption FromImage `
   -GalleryImageReference @{Id = $sourceImgVer.Id; Lun=1}
```

创建该磁盘。

```powershell
New-AzDisk -Disk $diskConfig `
   -ResourceGroupName $resourceGroup `
   -DiskName $diskName
```

## <a name="next-steps"></a>后续步骤

还可以使用 [Azure CLI](image-version-managed-image-cli.md) 或 [PowerShell](image-version-managed-image-powershell.md) 从托管磁盘创建映像版本。

<!-- Update_Description: new article about managed disk from image version -->
<!--NEW.date: 11/02/2020-->