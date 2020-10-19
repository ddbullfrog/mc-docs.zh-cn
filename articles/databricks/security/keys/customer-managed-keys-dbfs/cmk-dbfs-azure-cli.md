---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/15/2020
title: 使用 Azure CLI 为 DBFS 配置客户管理的密钥 - Azure Databricks
description: 了解如何使用 Azure CLI 配置自己的加密密钥以加密 DBFS 存储帐户。
ms.openlocfilehash: 648bfcb504c8cbe17b944ace5ede915040d48740
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937703"
---
# <a name="configure-customer-managed-keys-for-dbfs-using-the-azure-cli"></a>使用 Azure CLI 为 DBFS 配置客户管理的密钥

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 此功能仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中提供。

可以使用 Azure CLI 来配置自己的加密密钥以加密 DBFS 根存储帐户。 必须使用 [Azure Key Vault](/key-vault/general/overview) 存储密钥。

若要详细了解用于 DBFS 的客户管理的密钥，请参阅[为 DBFS 根配置客户管理的密钥](index.md)。

## <a name="install-the-azure-databricks-cli-extension"></a>安装 Azure Databricks CLI 扩展

1. [安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。
2. 安装 Azure Databricks CLI 扩展。

   ```bash
   az extension add --name databricks
   ```

## <a name="prepare-a-new-or-existing-azure-databricks-workspace-for-encryption"></a><a id="prepare-a-new-or-existing-azure-databricks-workspace-for-encryption"> </a><a id="prepare-cli"> </a>准备新的或现有的用于加密的 Azure Databricks 工作区

请将括号中的占位符值替换为你自己的值。 `<workspace-name>` 是 Azure 门户中显示的资源名称。

```bash
az login
az account set --subscription <subscription-id>
```

为创建工作区期间的加密做准备：

```bash
az databricks workspace create --name <workspace-name> --location <workspace-location> --resource-group <resource-group> --sku premium --prepare-encryption
```

准备用于加密的现有工作区：

```bash
az databricks workspace update --name <workspace-name> --resource-group <resource-group> --prepare-encryption
```

记下命令输出的 `storageAccountIdentity` 部分中的 `principalId` 字段。 配置密钥保管库时，需将其作为托管标识值提供。

有关 Azure Databricks 工作区的 Azure CLI 命令的详细信息，请参阅 [az databricks workspace 命令参考](https://docs.microsoft.com/cli/azure/ext/databricks/databricks/workspace)。

## <a name="create-a-new-key-vault"></a>创建新的 Key Vault

使用密钥保管库为根 DBFS 存储客户管理的密钥时，密钥保管库必须已启用两项密钥保护设置：“软删除”和“清除保护”。 若要新建启用了这些设置的密钥保管库，请运行以下命令。

请将括号中的占位符值替换为你自己的值。

```bash
az keyvault create \
        --name <key-vault> \
        --resource-group <resource-group> \
        --location <region> \
        --enable-soft-delete \
        --enable-purge-protection
```

若要详细了解如何使用 Azure CLI 启用“软删除”和“清除保护”，请参阅[如何将 Key Vault 软删除与 CLI 配合使用](/key-vault/general/soft-delete-cli)。

## <a name="configure-the-key-vault-access-policy"></a>配置 Key Vault 访问策略

使用 [az keyvault set-policy](https://docs.microsoft.com/cli/azure/keyvault?view=azure-cli-latest#az_keyvault_set_policy) 命令设置密钥保管库的访问策略，以便 Azure Databricks 工作区有权访问密钥保管库。

请将括号中的占位符值替换为你自己的值。

```bash
az keyvault set-policy \
        --name <key-vault> \
        --resource-group <resource-group> \
        --object-id <managed-identity>  \
        --key-permissions get unwrapKey wrapKey
```

将 `<managed-identity>` 替换为[准备用于加密的工作区](#prepare-cli)时记下的 `principalId` 值。

## <a name="create-a-new-key"></a>新建密钥

使用 [az keyvault key create](https://docs.microsoft.com/cli/azure/keyvault/key) 命令在密钥保管库中创建密钥。

请将括号中的占位符值替换为你自己的值。

```bash
az keyvault key create \
       --name <key> \
       --vault-name <key-vault>
```

DBFS 根存储支持 2048、3072 和 4096 大小的 RSA 和 RSA-HSM 密钥。 有关密钥的详细信息，请参阅[关于 Key Vault 密钥](/key-vault/keys/about-keys)。

## <a name="configure-dbfs-encryption-with-customer-managed-keys"></a>使用客户管理的密钥配置 DBFS 加密

将 Azure Databricks 工作区配置为使用在 Azure Key Vault 中创建的密钥。

请将括号中的占位符值替换为你自己的值。

```bash
key_vault_uri=$(az keyvault show \
 --name <key-vault> \
 --resource-group <resource-group> \
 --query properties.vaultUri \
--output tsv)
```

```bash
key_version=$(az keyvault key list-versions \
 --name <key> \ --vault-name <key-vault> \
 --query [-1].kid \
--output tsv | cut -d '/' -f 6)
```

```bash
az databricks workspace update --name <workspace-name> --resource-group <resource-group> --key-source Microsoft.KeyVault --key-name <key> --key-vault $key_vault_uri --key-version $key_version
```

## <a name="disable-customer-managed-keys"></a>禁用客户托管密钥

禁用客户托管密钥时，将再次使用 Microsoft 托管密钥对存储帐户进行加密。

请将括号中的占位符值替换为你自己的值，并使用在前面步骤中定义的变量。

```bash
az databricks workspace update --name <workspace-name> --resource-group <resource-group> --key-source Default
```