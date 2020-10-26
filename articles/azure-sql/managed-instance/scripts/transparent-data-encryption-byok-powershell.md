---
title: PowerShell：启用“创建自己的密钥”(BYOK) TDE
titleSuffix: Azure SQL Managed Instance
description: 了解如何配置 Azure SQL 托管实例，以开始使用创建自己的密钥 (BYOK) 透明数据加密 (TDE) 通过 PowerShell 进行静态加密。
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: WenJason
ms.author: v-jay
ms.reviewer: vanto, sstein
origin.date: 08/25/2020
ms.date: 10/29/2020
ms.openlocfilehash: fd85d0cdbca784fdab7976712b2ccc879bb2c1b7
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470459"
---
# <a name="transparent-data-encryption-in-sql-managed-instance-using-your-own-key-from-azure-key-vault"></a>使用 Azure Key Vault 中自己的密钥在 SQL 托管实例中实现透明数据加密

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

此 PowerShell 脚本实例使用 Azure Key Vault 中的密钥为 Azure SQL 托管实例配置使用客户托管密钥的透明数据加密 (TDE)。 这通常称为 TDE 的创建自己的密钥 (BYOK) 方案。 若要了解详细信息，请参阅[使用客户管理的密钥进行 Azure SQL 透明数据加密](../../database/transparent-data-encryption-byok-overview.md)。

## <a name="prerequisites"></a>先决条件

- 现有的托管实例。 请参阅[使用 PowerShell 创建托管实例](create-configure-managed-instance-powershell.md)。

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

使用 PowerShell 需要 Azure PowerShell 2.3.2 或更高版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-az-ps)，或运行以下示例脚本，以便为当前用户安装该模块：

`Install-Module -Name Az -AllowClobber -Scope CurrentUser`

此外，还需要运行 `Connect-AzAccount` 以创建与 Azure 的连接。

## <a name="sample-scripts"></a>示例脚本 

```powershell
# You will need an existing Managed Instance as a prerequisite for completing this script.
# See https://docs.azure.cn/sql-database/scripts/sql-database-create-configure-managed-instance-powershell

# Log in to your Azure account:
Connect-AzAccount -Environment AzureChinaCloud

# If there are multiple subscriptions, choose the one where AKV is created: 
Set-AzContext -SubscriptionId "subscription ID"

# Install the Az.Sql PowerShell package if you are running this PowerShell locally (uncomment below):
# Install-Module -Name Az.Sql

# 1. Create Resource and setup Azure Key Vault (skip if already done)

# Create Resource group (name the resource and specify the location)
$location = "chinaeast2" # specify the location
$resourcegroup = "MyRG" # specify a new RG name
New-AzResourceGroup -Name $resourcegroup -Location $location

# Create new Azure Key Vault with a globally unique VaultName and soft-delete option turned on:
$vaultname = "MyKeyVault" # specify a globally unique VaultName
New-AzKeyVault -VaultName $vaultname -ResourceGroupName $resourcegroup -Location $location

# Authorize Managed Instance to use the AKV (wrap/unwrap key and get public part of key, if public part exists): 
$objectid = (Set-AzSqlInstance -ResourceGroupName $resourcegroup -Name "MyManagedInstance" -AssignIdentity).Identity.PrincipalId
Set-AzKeyVaultAccessPolicy -BypassObjectIdValidation -VaultName $vaultname -ObjectId $objectid -PermissionsToKeys get,wrapKey,unwrapKey

# Allow access from trusted Azure services: 
Update-AzKeyVaultNetworkRuleSet -VaultName $vaultname -Bypass AzureServices

# Allow access from your client IP address(es) to be able to complete remaining steps: 
Update-AzKeyVaultNetworkRuleSet -VaultName $vaultname -IpAddressRange "xxx.xxx.xxx.xxx/xx"

# Turn the network rules ON by setting the default action to Deny: 
Update-AzKeyVaultNetworkRuleSet -VaultName $vaultname -DefaultAction Deny


# 2. Provide TDE Protector key (skip if already done)

# First, give yourself necessary permissions on the AKV, (specify your account instead of contoso.com):
Set-AzKeyVaultAccessPolicy -VaultName $vaultname -UserPrincipalName "myaccount@contoso.com" -PermissionsToKeys create,import,get,list

# The recommended way is to import an existing key from a .pfx file. Replace "<PFX private key password>" with the actual password below:
$keypath = "c:\some_path\mytdekey.pfx" # Supply your .pfx path and name
$securepfxpwd = ConvertTo-SecureString -String "<PFX private key password>" -AsPlainText -Force 
$key = Add-AzKeyVaultKey -VaultName $vaultname -Name "MyTDEKey" -KeyFilePath $keypath -KeyFilePassword $securepfxpwd

# ...or get an existing key from the vault:
# $key = Get-AzKeyVaultKey -VaultName $vaultname -Name "MyTDEKey"

# Alternatively, generate a new key directly in Azure Key Vault (recommended for test purposes only - uncomment below):
# $key = Add-AzureKeyVaultKey -VaultName $vaultname -Name MyTDEKey -Destination Software -Size 2048

# 3. Set up BYOK TDE on Managed Instance:

# Assign the key to the Managed Instance:
# $key = 'https://contoso.vault.azure.cn/keys/contosokey/01234567890123456789012345678901'
Add-AzSqlInstanceKeyVaultKey -KeyId $key.id -InstanceName "MyManagedInstance" -ResourceGroupName $resourcegroup

# Set TDE operation mode to BYOK: 
Set-AzSqlInstanceTransparentDataEncryptionProtector -Type AzureKeyVault -InstanceName "MyManagedInstance" -ResourceGroup $resourcegroup -KeyId $key.id
```

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/)。

SQL 托管实例的其他 PowerShell 脚本示例可在[ Azure SQL 托管实例 PowerShell 脚本](../../database/powershell-script-content-guide.md)中找到。
