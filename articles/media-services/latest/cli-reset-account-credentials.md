---
title: Azure CLI 脚本示例 - 重置帐户凭据 | Microsoft Docs
description: 使用 Azure CLI 脚本重置帐户凭据和恢复 app.config 设置。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: multiple
ms.workload: na
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1552d124bff176dec9c7bdd2c50bb8ea771ba00a
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245612"
---
# <a name="azure-cli-example-reset-the-account-credentials"></a>Azure CLI 示例：重置帐户凭据

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

本文中的 Azure CLI 脚本演示如何重置帐户凭据和恢复 app.config 设置。

## <a name="prerequisites"></a>先决条件

[创建媒体服务帐户](./create-account-howto.md)。

[!INCLUDE [media-services-cli-instructions.md](../../../includes/media-services-cli-instructions.md)]

## <a name="example-script"></a>示例脚本

```azurecli
# Update the following variables for your own settings:
$resourceGroup=amsResourceGroup
$amsAccountName=amsmediaaccountname

az ams account sp reset-credentials \
  --account-name $amsAccountName \
  --resource-group $resourceGroup
```

## <a name="next-steps"></a>后续步骤

* [az ams](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
* [重置凭据](https://docs.microsoft.com/cli/azure/ams/account/sp?view=azure-cli-latest#az-ams-account-sp-reset-credentials)
