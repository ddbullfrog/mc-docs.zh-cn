---
title: 使用 Azure CLI 将内容上传到 Azure 媒体服务资产
description: 本主题中的 Azure CLI 脚本演示如何创建媒体服务资产供上传内容使用。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.workload: na
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.custom: devx-track-azurecli
ms.openlocfilehash: 02543b36c5815dacff36c6e3d07ad4b3ba6640a6
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245648"
---
# <a name="create-an-asset"></a>创建资产

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

本文介绍如何创建媒体服务资产。  你将使用资产来保存用于编码和流式传输的媒体内容。  若要了解有关媒体服务资产的详细信息，请参阅 [Azure 媒体服务 v3 中的资产](assets-concept.md)

## <a name="prerequisites"></a>先决条件

按照[创建媒体服务帐户](./create-account-howto.md)中的步骤操作，创建所需的媒体服务帐户和资源组以创建资产。

## <a name="methods"></a>方法

## <a name="cli"></a>[CLI](#tab/cli/)

[!INCLUDE [Create an asset with CLI](./includes/task-create-asset-cli.md)]

## <a name="example-script"></a>示例脚本

```azurecli
#!/bin/bash

# Update the following variables for your own settings:
$resourceGroup=amsResourceGroup
$amsAccountName=amsmediaaccountname
$assetName="myAsset-uniqueID"
$expiry=$(date -u +"%Y-%m-%dT%TZ" -d "+23 hours")

# Create a Media Services Asset to upload content to.
# In the v3 API, Asset names are unique ARM resource identifiers and must be unique to the account.
# It's recommended to use short unique IDs or GUIDs to keep the names unique to the account.
az ams asset create \
    -n $assetName \
    -a $amsAccountName \
    -g $resourceGroup \

# Get the SAS URLs to upload content to the container for the Asset
# Default is 23 hour expiration, but you can adjust with the --expiry flag. 
# Max supported is 24 hours. 
az ams asset get-sas-urls \
    -n $assetName \
    -a $amsAccountName \
    -g $resourceGroup \
    --expiry  $expiry\
    --permissions ReadWrite \

# Use the az storage modules to upload a local file to the container using the SAS URL from previous step.
# If you are logged in already to the subscription with access to the storage account, you do not need to use the --sas-token at all. Just eliminate it below.
# The container name is in the SAS URL path, and should be set with the -c option.
# Use the -f option to point to a local file on your machine.
# Use the -n option to name the blob in storage.
# Use the --account-name option to point to the storage account name to use 
# Use the --sas-token option to place the SAS token after the query string from previous step. 
# NOTE that the SAS Token is only good for up to 24 hours max. 

#   az storage blob upload \
#       -c asset-84045780-a71c-4511-801b-711b1a2e76b2 \
#       -f C:\Videos\ignite-short.mp4 \
#       -n ignite-short.mp4 \
#       --account-name mconverticlitest0003 \
#       --sas-token "?sv=2015-07-08&sr=c&sig=BvMXDCOjR%2FOP2%2FYi6lVknC4Gcq7fIun5tst8jgED7zY%3D&se=2018-04-25T00:00:00Z&sp=rwl" \

echo "press  [ENTER]  to continue."
read continue
```

## <a name="rest"></a>[REST](#tab/rest/)

### <a name="using-rest"></a>使用 REST

[!INCLUDE [media-services-cli-instructions.md](./includes/task-create-asset-rest.md)]

### <a name="using-curl"></a>使用 cURL

[!INCLUDE [media-services-cli-instructions.md](./includes/task-create-asset-curl.md)]

## <a name="using-postman"></a>使用 Postman

[!INCLUDE[Create an asset with Postman](./includes/task-create-asset-postman.md)]

## <a name="net"></a>[.NET](#tab/net/)

[!INCLUDE [media-services-cli-instructions.md](./includes/task-create-asset-dotnet.md)]

---

## <a name="next-steps"></a>后续步骤

[媒体服务概述](media-services-overview.md)
