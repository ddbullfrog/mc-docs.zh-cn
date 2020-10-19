---
title: 在租户之间共享库映像
description: 了解如何使用通过共享映像库在 Azure 租户之间共享的映像创建规模集。
author: cynthn
ms.author: v-junlch
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.date: 09/28/2020
ms.reviewer: akjosh
ms.custom: akjosh, devx-track-azurecli
ms.openlocfilehash: 436bf677d193a8422cc14e058c583068be3c1f5d
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937280"
---
# <a name="share-images-across-tenants-with-shared-image-gallery"></a>使用共享映像库在租户之间共享映像

[!INCLUDE [virtual-machines-share-images-across-tenants](../../includes/virtual-machines-share-images-across-tenants.md)]


## <a name="create-a-scale-set-using-azure-cli"></a>使用 Azure CLI 创建规模集

使用租户 1 的 appID、应用密钥以及 ID 登录到租户 1 的服务主体。 可以根据需要使用 `az account show --query "tenantId"` 获取租户 ID。

```azurecli
az account clear
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 1 ID>'
az account get-access-token 
```
 
使用租户 2 的 appID、应用密钥以及 ID 登录到租户 2 的服务主体：

```azurecli
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 2 ID>'
az account get-access-token
```

创建规模集。 请将示例中的信息替换为你自己的。

```azurecli
az vmss create \
  -g myResourceGroup \
  -n myScaleSet \
  --image "/subscriptions/<Tenant 1 subscription>/resourceGroups/<Resource group>/providers/Microsoft.Compute/galleries/<Gallery>/images/<Image definition>/versions/<version>" \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="next-steps"></a>后续步骤

如果遇到任何问题，可以[对共享映像库进行故障排除](../virtual-machines/troubleshooting-shared-images.md)。

