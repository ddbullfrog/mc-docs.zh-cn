---
title: Azure CLI Script 示例 - 加密 Linux VM
description: Azure CLI 脚本示例 - 加密 Linux VM
services: virtual-machines-linux
documentationcenter: virtual-machines
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
origin.date: 12/15/2017
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: fa847d0fa8656bd76f8044953659eb1b5dee511c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105114"
---
# <a name="encrypt-a-linux-virtual-machine-in-azure"></a>在 Azure 中加密 Linux 虚拟机

此脚本创建安全的 Azure 密钥保管库、加密密钥、Azure Active Directory 服务主体和 Linux 虚拟机 (VM)。 然后使用 Key Vault 和服务主体凭据中的加密密钥对 VM 进行加密。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/bash

# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login

# Provide your own unique Key Vault name
keyvault_name=<your_unique_keyvault_name>

# Register the Key Vault provider and create a resource group.
az provider register -n Microsoft.KeyVault
az group create --name myResourceGroup --location chinaeast

# Create a Key Vault for storing keys and enabled for disk encryption.
az keyvault create --name $keyvault_name --resource-group myResourceGroup --location chinaeast \
    --enabled-for-disk-encryption True

# Create a key within the Key Vault.
az keyvault key create --vault-name $keyvault_name --name myKey --protection software

# Create an Azure Active Directory service principal for authenticating requests to Key Vault.
# Read in the service principal ID and password for use in later commands.
read sp_id sp_password <<< $(az ad sp create-for-rbac --query [appId,password] -o tsv)

# Grant permissions on the Key Vault to the AAD service principal.
az keyvault set-policy --name $keyvault_name --spn $sp_id \
    --key-permissions wrapKey \
    --secret-permissions set

# Create a virtual machine.
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image OpenLogic:CentOS:7.2n:7.2.20160629 \
    --admin-username azureuser \
    --generate-ssh-keys

# Encrypt the VM disks.
az vm encryption enable --resource-group myResourceGroup --name myVM \
  --aad-client-id $sp_id \
  --aad-client-secret $sp_password \
  --disk-encryption-keyvault $keyvault_name \
  --key-encryption-key myKey \
  --volume-type all

# Output how to monitor the encryption status and next steps.
echo "The encryption process can take some time. View status with:

    az vm encryption show --resource-group myResourceGroup --name myVM --query [osDisk] -o tsv

When encryption status shows \`VMRestartPending\`, restart the VM with:

    az vm restart --resource-group myResourceGroup --name myVM"

```

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

```azurecli
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、Azure 密钥保管库、服务主体、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 注释 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az keyvault create](https://docs.azure.cn/cli/keyvault?view=azure-cli-latest#az-keyvault-create) | 创建 Azure Key Vault，存储加密密钥等安全数据。 |
| [az keyvault key create](https://docs.azure.cn/cli/keyvault/key?view=azure-cli-latest#az-keyvault-key-create) | 在 Key Vault 中创建加密密钥。 |
| [az ad sp create-for-rbac](https://docs.azure.cn/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) | 创建 Azure Active Directory 服务主体，安全地进行身份验证并控制对加密密钥的访问。 |
| [az keyvault set-policy](https://docs.azure.cn/cli/keyvault?view=azure-cli-latest#az-keyvault-set-policy) | 设置对 Key Vault 的权限，授予服务主体访问加密密钥的权限。 |
| [az vm create](https://docs.azure.cn/cli/vm?view=azure-cli-latest#az-vm-create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az vm encryption enable](https://docs.azure.cn/cli/vm/encryption?view=azure-cli-latest#az-vm-encryption-enable) | 使用服务主体凭据和加密密钥对 VM 进行加密。 |
| [az vm encryption show](https://docs.azure.cn/cli/vm/encryption?view=azure-cli-latest#az-vm-encryption-show) | 显示 VM 加密过程的状态。 |
| [az group delete](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-delete) | 删除资源组，包括所有嵌套的资源。 |

<!--CORRECT ON [az group delete](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-delete)-->

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.azure.cn/cli/index?view=azure-cli-latest)。

可以在 [Azure Linux VM 文档](../linux/cli-samples.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)中找到其他虚拟机 CLI 脚本示例。

<!--Update_Description: update meta properties, update link -->