---
title: 使用 Azure CLI 从专用化映像版本创建 VM
description: 使用 Azure CLI 从共享映像库中的专用化映像版本创建 VM。
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
origin.date: 04/23/2020
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: ffafa5a3a4dbe44c19494a531b794ecead3431ce
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106128"
---
<!--Verified successfully-->
# <a name="create-a-vm-using-a-specialized-image-version-with-the-azure-cli"></a>使用 Azure CLI 通过专用化映像版本创建 VM

从共享映像库中存储的[专用化映像版本](./linux/shared-image-galleries.md#generalized-and-specialized-images)创建 VM。 若要使用通用化映像版本创建 VM，请参阅[从通用化映像版本创建 VM](vm-generalized-image-version-cli.md)。

在此示例中，请根据需要替换资源名称。 

使用 [az sig image-definition list](https://docs.microsoft.com/cli/azure/sig?view=azure-cli-latest#az-sig-image-definition-list) 列出库中的映像定义，以查看定义的名称和 ID。

```azurecli 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

结合 --specialized 参数使用 [az vm create](https://docs.azure.cn/cli/vm?view=azure-cli-latest#az-vm-create) 创建 VM 可以指明该映像是专用化映像。 

使用 `--image` 的映像定义 ID 从可用的最新映像版本创建 VM。 还可以通过为 `--image` 提供映像版本 ID 从特定版本创建 VM。 

在此示例中，我们将从 myImageDefinition 映像的最新版本创建 VM。

```azurecli
az group create --name myResourceGroup --location chinaeast
az vm create --resource-group myResourceGroup \
    --name myVM \
    --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
    --specialized
```

## <a name="next-steps"></a>后续步骤

<!--Not Available on [Azure Image Builder (preview)](./windows/image-builder-overview.md)-->
<!--Not Available on [create a new image version from an existing image version](./linux/image-builder-gallery-update-image-version.md)-->

此外可以使用模板创建共享映像库资源。 提供多个 Azure 快速入门模板： 

- [创建共享映像库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sig-create/)
- [在共享的映像库中创建映像定义](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sig-image-definition-create/)
- [在共享映像库中创建映像版本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sig-image-version-create/)
- [根据映像版本创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-sig/)

<!-- Update_Description: update meta properties, wording update, update link -->