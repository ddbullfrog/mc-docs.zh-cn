---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/15/2020
title: 使用 PowerShell 为 DBFS 配置客户管理的密钥 - Azure Databricks
description: 了解如何使用 PowerShell 配置自己的加密密钥以加密 DBFS 存储帐户。
ms.openlocfilehash: 94f9dec73d8b98ad4aafd14706a108b2f6a7e872
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937719"
---
# <a name="configure-customer-managed-keys-for-dbfs-using-powershell"></a>使用 PowerShell 为 DBFS 配置客户管理的密钥

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 此功能仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中提供。

可以使用 PowerShell 配置自己的加密密钥以加密 DBFS 根存储帐户。 必须使用 [Azure Key Vault](/key-vault/general/overview) 存储密钥。

若要详细了解用于 DBFS 的客户管理的密钥，请参阅[为 DBFS 根配置客户管理的密钥](index.md)。

## <a name="install-the-azure-databricks-powershell-module"></a>安装 Azure Databricks PowerShell 模块

1. [安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)。
2. [安装 Azure Databricks PowerShell 模块](https://www.powershellgallery.com/packages/Az.Databricks/0.1.1)。

## <a name="prepare-a-new-or-existing-azure-databricks-workspace-for-encryption"></a>准备新的或现有的用于加密的 Azure Databricks 工作区

请将括号中的占位符值替换为你自己的值。 `<workspace-name>` 是 Azure 门户中显示的资源名称。

在创建工作区时准备加密：

```powershell
$workSpace = New-AzDatabricksWorkspace -Name <workspace-name> -Location <workspace-location> -ResourceGroupName <resource-group> -Sku premium -PrepareEncryption
```

准备用于加密的现有工作区：

```powershell
$workSpace = Update-AzDatabricksWorkspace -Name <workspace-name> -ResourceGroupName <resource-group> -PrepareEncryption
```

有关 Azure Databricks 工作区的 PowerShell cmdlet 的详细信息，请参阅 [Az.Databricks 参考](https://docs.microsoft.com/powershell/module/az.databricks)。

## <a name="create-a-new-key-vault"></a>创建新的 Key Vault

使用密钥保管库为默认的（根）DBFS 存储客户管理的密钥时，密钥保管库必须已启用两项密钥保护设置：“软删除”和“清除保护”。

在 `Az.KeyVault` 模块的版本 2.0.0 及更高版本中，当创建新密钥保管库时，默认会启用软删除。

以下示例在启用了“软删除”和“清除保护”属性的情况下创建新密钥保管库。 请将括号中的占位符值替换为你自己的值。

```powershell
$keyVault = New-AzKeyVault -Name <key-vault> `
     -ResourceGroupName <resource-group> `
     -Location <location> `
     -EnablePurgeProtection
```

若要了解如何使用 PowerShell 在现有密钥保管库上启用“软删除”和“清除保护”，请参阅[如何在 PowerShell 中使用 Key Vault 软删除](/key-vault/general/soft-delete-powershell)中的“启用软删除”和“启用清除保护”。

### <a name="configure-the-key-vault-access-policy"></a>配置 Key Vault 访问策略

使用 [Set-AzKeyVaultAccessPolicy](https://docs.microsoft.com/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) 设置密钥保管库的访问策略，以便 Azure Databricks 工作区有权访问密钥保管库。

```powershell
Set-AzKeyVaultAccessPolicy `
      -VaultName $keyVault.VaultName `
      -ObjectId $workSpace.StorageAccountIdentity.PrincipalId `
      -PermissionsToKeys wrapkey,unwrapkey,get
```

## <a name="create-a-new-key"></a>新建密钥

使用 [Add-AzKeyVaultKey](https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultkey) cmdlet 在密钥保管库中创建新密钥。 请将括号中的占位符值替换为你自己的值。

```powershell
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName -Name <key> -Destination 'Software'
```

DBFS 根存储支持 2048、3072 和 4096 大小的 RSA 和 RSA-HSM 密钥。 有关密钥的详细信息，请参阅[关于 Key Vault 密钥](/key-vault/keys/about-keys)。

## <a name="configure-dbfs-encryption-with-customer-managed-keys"></a>使用客户管理的密钥配置 DBFS 加密

将 Azure Databricks 工作区配置为使用在 Azure Key Vault 中创建的密钥。 请将括号中的占位符值替换为你自己的值。

```powershell
Update-AzDatabricksWorkspace -ResourceGroupName <resource-group> `
      -Name <workspace-name>
     -EncryptionKeySource Microsoft.Keyvault `
     -EncryptionKeyName $key.Name `
     -EncryptionKeyVersion $key.Version `
     -EncryptionKeyVaultUri $keyVault.VaultUri
```

## <a name="disable-customer-managed-keys"></a>禁用客户托管密钥

禁用客户托管密钥时，将再次使用 Microsoft 托管密钥对存储帐户进行加密。

请将括号中的占位符值替换为你自己的值，并使用在前面步骤中定义的变量。

```powershell
Update-AzDatabricksWorkspace -Name <workspace-name> -ResourceGroupName <resource-group> -EncryptionKeySource Default
```