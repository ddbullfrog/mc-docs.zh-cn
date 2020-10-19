---
title: 使用 Azure CLI 从通用化映像创建规模集
description: 借助 Azure CLI 使用共享映像库中的通用化映像创建规模集。
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 09/28/2020
ms.author: v-junlch
ms.reviewer: akjosh
ms.openlocfilehash: 1fe01a888ac5e1175f37c90b9c92d6613a73c794
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937281"
---
# <a name="create-a-scale-set-from-a-generalized-image-with-azure-cli"></a>使用 Azure CLI 从通用化映像创建规模集

使用 Azure CLI 从[共享映像库](shared-image-galleries.md)中存储的通用化映像版本创建规模集。 若要使用专用化映像版本创建规模集，请参阅[从专用化映像创建规模集实例](instance-specialized-image-version-cli.md)。

如果选择在本地安装并使用 CLI，本教程要求运行 Azure CLI 2.4.0 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

在此示例中，请根据需要替换资源名称。 

使用 [az sig image-definition list](https://docs.microsoft.com/en-us/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-list) 列出库中的映像定义，以查看定义的名称和 ID。

```azurecli 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

使用 [`az vmss create`](/cli/vmss#az-vmss-create) 创建规模集。 

使用 `--image` 的映像定义 ID 从可用的最新映像版本创建规模集实例。 还可以通过提供 `--image` 的映像版本 ID 从特定版本创建规模集实例。 请注意，使用特定映像版本意味着：如果该特定映像版本由于已删除或已从区域中删除而无法使用，则自动化可能会失败。 建议使用映像定义 ID 来创建新的 VM（除非需要特定的映像版本）。

在此示例中，我们将从 myImageDefinition 映像的最新版本创建实例。

```azurecli
az group create --name myResourceGroup --location chinanorth2
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" 
   --admin-username azureuser \
   --generate-ssh-keys
```

创建和配置所有的规模集资源和 VM 需要几分钟时间。

## <a name="next-steps"></a>后续步骤

此外可以使用模板创建共享映像库资源。 提供多个 Azure 快速入门模板： 

- [创建共享映像库](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [在共享的映像库中创建映像定义](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [在共享映像库中创建映像版本](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)

有关共享映像库的详细信息，请参阅[概述](shared-image-galleries.md)。 如果遇到问题，请参阅[排查共享映像库问题](../virtual-machines/troubleshooting-shared-images.md)。

