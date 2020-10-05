---
title: 使用 Azure CLI 配置客户管理的密钥
titleSuffix: Azure Storage
description: 了解如何使用 Azure CLI 通过 Azure Key Vault 来配置客户管理的密钥用于 Azure 存储加密。
services: storage
author: WenJason
ms.service: storage
ms.topic: how-to
origin.date: 08/24/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7f5cfe5279db425f4dd2117a25c9d73c8afabf45
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026509"
---
# <a name="configure-customer-managed-keys-with-azure-key-vault-by-using-azure-cli"></a>通过 Azure CLI 使用 Azure Key Vault 配置客户管理的密钥

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

本文介绍如何使用 Azure CLI 配置包含客户管理的密钥的 Azure Key Vault。 要了解如何使用 Azure CLI 创建密钥保管库，请参阅[快速入门：使用 Azure CLI 在 Azure Key Vault 中设置和检索机密](../../key-vault/secrets/quick-create-cli.md)。

## <a name="assign-an-identity-to-the-storage-account"></a>将标识分配到存储帐户

若要为存储帐户启用客户管理的密钥，请先将一个系统分配的托管标识分配到该存储帐户。 将使用此托管标识授予存储帐户访问 Key Vault 的权限。

若要使用 Azure CLI 分配托管标识，请调用 [az storage account update](/cli/storage/account#az-storage-account-update)。 请记得将括号中的占位符值替换为你自己的值。

```azurecli
az login
az account set --subscription <subscription-id>

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

有关使用 Azure CLI 配置系统分配的托管标识的详细信息，请参阅[使用 Azure CLI 在 Azure VM 上配置 Azure 资源的托管标识](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)。

## <a name="create-a-new-key-vault"></a>创建新的 Key Vault

必须为用来存储客户管理的密钥（用于 Azure 存储加密）的 Key Vault 启用两项密钥保护设置：“软删除”和“不要清除”。  若要在启用这些设置的情况下使用 PowerShell 或 Azure CLI 创建新的 Key Vault，请执行以下命令。 请记得将括号中的占位符值替换为你自己的值。

若要使用 Azure CLI 创建新的 Key Vault，请调用 [az keyvault create](/cli/keyvault#az-keyvault-create)。 请记得将括号中的占位符值替换为你自己的值。

```azurecli
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-soft-delete \
    --enable-purge-protection
```

若要了解如何使用 Azure CLI 在现有密钥保管库上启用“软删除”和“请勿清除”，请参阅[如何在 CLI 中使用软删除](../../key-vault/general/soft-delete-cli.md)中标题为“启用软删除”和“启用清除保护”的部分。

## <a name="configure-the-key-vault-access-policy"></a>配置 Key Vault 访问策略

接下来，配置 Key Vault 的访问策略，使存储帐户有权访问 Key Vault。 此步骤使用前面分配给存储帐户的托管标识。

若要设置 Key Vault 的访问策略，请调用 [az keyvault set-policy](/cli/keyvault#az-keyvault-set-policy)。 请记得将括号中的占位符值替换为你自己的值。

```azurecli
storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

## <a name="create-a-new-key"></a>新建密钥

接下来，在 Key Vault 中创建密钥。 若要创建密钥，请调用 [az keyvault key create](/cli/keyvault/key#az-keyvault-key-create)。 请记得将括号中的占位符值替换为你自己的值。

```azurecli
az keyvault key create \
    --name <key> \
    --vault-name <key-vault>
```

Azure 存储加密支持 2048、3072 和 4096 大小的 RSA 密钥。 有关密钥的详细信息，请参阅[关于 Azure Key Vault 密钥、机密和证书](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys)中的“Key Vault 密钥”。

## <a name="configure-encryption-with-customer-managed-keys"></a>配置使用客户管理的密钥进行加密

Azure 存储加密默认使用 Microsoft 托管的密钥。 在这一步中，请将 Azure 存储帐户配置为通过 Azure Key Vault 使用客户管理的密钥，然后指定要与存储帐户关联的密钥。

在使用客户管理的密钥配置加密时，可以选择在关联的密钥保管库中的密钥版本发生更改时自动更新用于加密的密钥。 也可显式指定在手动更新密钥版本之前用于加密的密钥版本。

> [!NOTE]
> 若要轮换密钥，请在 Azure Key Vault 中创建密钥的新版本。 Azure 存储不会处理 Azure Key Vault 中的密钥轮换，因此你需要手动轮换密钥，或创建一个函数以便按计划轮换密钥。

### <a name="configure-encryption-to-automatically-update-the-key-version"></a>配置加密以自动更新密钥版本

若要使用客户管理的密钥配置加密以自动更新密钥版本，请安装 [Azure CLI 2.4.0](/cli/release-notes-azure-cli#april-21-2020) 或更高版本。 有关详细信息，请参阅[安装 Azure CLI](/cli/install-azure-cli?view=azure-cli-latest)。

若要自动更新客户管理的密钥的密钥版本，请在使用存储帐户的客户管理的密钥配置加密时省略密钥版本。 请调用 [az storage account update](/cli/storage/account#az-storage-account-update)，以便更新存储帐户的加密设置，如以下示例所示。 包括 `--encryption-key-source` 参数并将其设置为 `Microsoft.Keyvault` 即可为帐户启用客户管理的密钥。 请记得将括号中的占位符值替换为你自己的值。

```azurecli
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

### <a name="configure-encryption-for-manual-updating-of-key-versions"></a>为手动更新密钥版本配置加密

若要显式指定用于加密的密钥版本，请在为存储帐户配置使用客户管理的密钥进行的加密时提供该密钥版本。 请调用 [az storage account update](/cli/storage/account#az-storage-account-update)，以便更新存储帐户的加密设置，如以下示例所示。 包括 `--encryption-key-source` 参数并将其设置为 `Microsoft.Keyvault` 即可为帐户启用客户管理的密钥。 请记得将括号中的占位符值替换为你自己的值。

```azurecli
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [-1].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

手动更新密钥版本时，需要更新存储帐户的加密设置以使用新版本。 首先，通过调用 [az keyvault show](/cli/keyvault#az-keyvault-show) 查询 Key Vault URI，并通过调用 [az keyvault key list-versions](/cli/keyvault/key#az-keyvault-key-list-versions) 查询密钥版本。 然后调用 [az storage account update](/cli/storage/account#az-storage-account-update) 来更新存储帐户的加密设置，以使用新的密钥版本，如上一示例所示。

## <a name="use-a-different-key"></a>使用其他密钥

若要更改用于 Azure 存储加密的密钥，请调用 [az storage account update](/cli/storage/account#az-storage-account-update)（如[使用客户托管密钥配置加密](#configure-encryption-with-customer-managed-keys)中所示），并提供新的密钥名称和版本。 如果新密钥位于不同的密钥保管库中，还需要更新密钥保管库 URI。

## <a name="revoke-customer-managed-keys"></a>撤销客户托管密钥

可以通过删除密钥保管库访问策略来撤销客户管理的密钥。 若要撤销客户托管密钥，请调用 [az keyvault delete-policy](/cli/keyvault#az-keyvault-delete-policy) 命令，如下例所示。 请记得将括号中的占位符值替换为自己的值，并使用前面示例中定义的变量。

```azurecli
az keyvault delete-policy \
    --name <key-vault> \
    --object-id $storage_account_principal
```

## <a name="disable-customer-managed-keys"></a>禁用客户托管密钥

禁用客户托管密钥时，将再次使用 Microsoft 托管密钥对存储帐户进行加密。 若要禁用客户托管密钥，请调用 [az storage account update](/cli/storage/account#az-storage-account-update) 并将 `--encryption-key-source parameter` 设置为 `Microsoft.Storage`，如以下示例所示。 请记得将括号中的占位符值替换为自己的值，并使用前面示例中定义的变量。

```azurecli
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-source Microsoft.Storage
```

## <a name="next-steps"></a>后续步骤

- [静态数据的 Azure 存储加密](storage-service-encryption.md) 
- [什么是 Azure Key Vault？](/key-vault/key-vault-overview)
