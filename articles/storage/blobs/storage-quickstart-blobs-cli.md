---
title: 快速入门 - 使用 Azure CLI 创建 blob
titleSuffix: Azure Storage
description: 在本快速入门中，你将了解如何使用 Azure CLI 将 blob 上传到 Azure 存储、下载 blob 以及在容器中列出 blob。
services: storage
author: WenJason
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
origin.date: 08/17/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.custom: devx-track-azurecli
ms.openlocfilehash: 455fe1743a37d00bea7b640c13ee0995a8be5195
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026486"
---
# <a name="quickstart-create-download-and-list-blobs-with-azure-cli"></a>快速入门：使用 Azure CLI 创建、下载和列出 blob

Azure CLI 是 Azure 的命令行体验，用于管理 Azure 资源。 可以将其安装在 macOS、Linux 和 Windows 上，然后从命令行运行它。 本快速入门介绍了如何使用 Azure CLI 通过 Azure Blob 存储来上传和下载数据。

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

本快速入门需要运行 Azure CLI 2.0.46 或更高版本。 运行 `az --version` 即可确定你的版本。 如需进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

若要登录到 Azure CLI，请在浏览器窗口中运行 `az login` 并进行身份验证：

```azurecli
az login
```

有关使用 Azure CLI 进行身份验证的详细信息，请参阅[使用 Azure CLI 登录](/cli/authenticate-azure-cli)。

## <a name="authorize-access-to-blob-storage"></a>授予对 Blob 存储的访问权限

可以使用 Azure AD 凭据或存储帐户访问密钥通过 Azure CLI 授予对 Blob 存储的访问权限。 建议使用 Azure AD 凭据。 本文介绍如何使用 Azure AD 授权 Blob 存储操作。

与针对 Blob 存储的数据操作相对应的 Azure CLI 命令支持 `--auth-mode` 参数，该参数用于指定如何授权给定操作。 将 `--auth-mode` 参数设置为 `login`，使用 Azure AD 凭据进行授权。 有关详细信息，请参阅[使用 Azure CLI 授权访问 blob 或队列数据](../common/authorize-data-operations-cli.md?toc=%2fstorage%2fblobs%2ftoc.json)。

仅 Blob 存储数据操作支持 `--auth-mode` 参数。 管理操作（例如创建资源组或存储帐户）会自动将 Azure AD 凭据用于授权。

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](/cli/group) 命令创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。

请务必将尖括号中的占位符值替换为你自己的值：

```azurecli
az group create \
    --name <resource-group> \
    --location <location>
```

## <a name="create-a-storage-account"></a>创建存储帐户

使用 [az storage account create](/cli/storage/account) 命令创建常规用途存储帐户。 常规用途的存储帐户可用于以下四种服务：Blob、文件、表和队列。

请务必将尖括号中的占位符值替换为你自己的值：

```azurecli
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_GRS \
    --encryption-services blob
```

## <a name="create-a-container"></a>创建容器

始终将 Blob 上传到容器中。 可以在容器中整理 Blob 组，就像在计算机的文件夹中整理文件一样。 可以使用 [az storage container create](/cli/storage/container) 命令创建用于存储 blob 的容器。 

下面的示例使用 Azure AD 帐户授权操作创建容器。 创建容器之前，请将[存储 Blob 数据参与者](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)角色分配给自己。 即使你是帐户所有者，也需要显式权限才能对存储帐户执行数据操作。 有关分配 Azure 角色的详细信息，请参阅[使用 Azure CLI 为访问分配 Azure 角色](../common/storage-auth-aad-rbac-cli.md?toc=/storage/blobs/toc.json)。  

请务必将尖括号中的占位符值替换为你自己的值：

```azurecli
az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"

az storage container create \
    --account-name <storage-account> \
    --name <container> \
    --auth-mode login
```

> [!IMPORTANT]
> Azure 角色分配可能需要几分钟时间来进行传播。

你还可以使用存储帐户密钥来授权操作创建容器。 有关使用 Azure CLI 授权数据操作的详细信息，请参阅[使用 Azure CLI 授权访问 blob 或队列数据](../common/authorize-data-operations-cli.md?toc=/storage/blobs/toc.json)。

## <a name="upload-a-blob"></a>上传 blob

Blob 存储支持块 blob、追加 blob 和页 blob。 本快速入门中的示例介绍如何使用块 blob。

首先，创建要上传到块 blob 的文件。

此示例使用 [az storage blob upload](/cli/storage/blob) 命令将 Blob 上传到在上一个步骤中创建的容器中。

```azurecli
az storage blob upload \
    --account-name <storage-account> \
    --container-name <container> \
    --name helloworld \
    --file ~/path/to/local/file \
    --auth-mode login
```

此操作将创建 Blob（如果该 Blob 尚不存在），或者覆盖 Blob（如果该 Blob 已存在）。 上传尽可能多的文件，然后继续操作。

若要同时上传多个文件，则可使用 [az storage blob upload-batch](/cli/storage/blob) 命令。

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

使用 [az storage blob list](/cli/storage/blob) 命令列出容器中的 blob。 请务必将尖括号中的占位符值替换为你自己的值：

```azurecli
az storage blob list \
    --account-name <storage-account> \
    --container-name <container> \
    --output table \
    --auth-mode login
```

## <a name="download-a-blob"></a>下载 Blob

使用 [az storage blob download](/cli/storage/blob) 命令下载之前上传的 Blob。 请务必将尖括号中的占位符值替换为你自己的值：

```azurecli
az storage blob download \
    --account-name <storage-account> \
    --container-name <container> \
    --name helloworld \
    --file ~/destination/path/for/file \
    --auth-mode login
```

## <a name="data-transfer-with-azcopy"></a>使用 AzCopy 传输数据

AzCopy 命令行实用程序提供适用于 Azure 存储的高性能且可编写脚本的数据传输。 可使用 AzCopy 将数据传输到 Blob 存储和 Azure 文件存储，或将数据从其中传出。 有关 AzCopy v10（最新版 AzCopy）的详细信息，请参阅 [AzCopy 入门](../common/storage-use-azcopy-v10.md)。 若要了解如何将 AzCopy v10 与 Blob 存储配合使用，请参阅[使用 AzCopy 和 Blob 存储传输数据](../common/storage-use-azcopy-blobs.md)。

以下示例使用 AzCopy 将本地文件上传到 blob。 请务必将示例值替换为你自己的值：

```bash
azcopy login --aad-endpoint https://login.partner.microsoftonline.cn
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myTextFile.txt'
```

## <a name="clean-up-resources"></a>清理资源

若要删除在本快速入门中创建的资源（包括存储帐户），请使用 [az group delete](/cli/group) 命令删除资源组。 请务必将尖括号中的占位符值替换为你自己的值：

```azurecli
az group delete \
    --name <resource-group> \
    --no-wait
```

## <a name="next-steps"></a>后续步骤

在此快速入门中，你了解了如何在本地文件系统和 Azure Blob 存储中的容器之间传输文件。 若要详细了解如何使用 Azure CLI 操作 Blob 存储，请浏览“适用于 Blob 存储的 Azure CLI 示例”。

> [!div class="nextstepaction"]
> [适用于 Blob 存储的 Azure CLI 示例](/storage/blobs/storage-samples-blobs-cli?toc=%2fstorage%2fblobs%2ftoc.json)
