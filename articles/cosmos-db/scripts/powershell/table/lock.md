---
title: 用于为 Azure Cosmos 表 API 表创建资源锁的 PowerShell 脚本
description: 为 Azure Cosmos 表 API 表创建资源锁
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
origin.date: 06/12/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 08/17/2020
ms.author: v-yeche
ms.openlocfilehash: 6ac67f6f46cdce33c5e6fc70d4041696a2150438
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327449"
---
<!--Verified successfully-->
# <a name="create-a-resource-lock-for-azure-cosmos-table-api-table-using-azure-powershell"></a>使用 Azure PowerShell 为 Azure Cosmos 表 API 表创建资源锁
[!INCLUDE[appliesto-table-api](../../../includes/appliesto-table-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-PowerShell-install](../../../../../includes/sample-PowerShell-install-no-ssh.md)]

> [!IMPORTANT]
> 除非在启用了 `disableKeyBasedMetadataWriteAccess` 属性的情况下先锁定 Cosmos DB 帐户，否则资源锁对使用任何 Cosmos DB SDK、任何工具（通过帐户密钥连接）或 Azure 门户进行连接的用户所作的更改均无效。 要详细了解如何启用此属性，请参阅[防止 SDK 更改](../../../role-based-access-control.md#prevent-sdk-changes)。

## <a name="sample-script"></a>示例脚本

```PowerShell
# References:
# Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# Az.Resources | https://docs.microsoft.com/powershell/module/az.resources
# --------------------------------------------------
# Purpose
# 
# --------------------------------------------------
# Variables
# ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup"
$accountName = "myaccount"
$databaseName = "myDatabase"
# *****
$apiVersion = "2020-04-01" # Cosmos DB RP API version
$lockLevel = "CanNotDelete" # CanNotDelete or ReadOnly

$resourceTypeAccount = "Microsoft.DocumentDB/databaseAccounts"
$resourceTypeDatabase = "$resourceTypeAccount/tables"

$resourceNameDatabase = "$accountName/$databaseName"
$lockNameDatabase = "$accountName-$databaseName-Lock"
# --------------------------------------------------

Write-Host "Create a $lockLevel lock on resource $resourceNameDatabase"
New-AzResourceLock `
    -ApiVersion $apiVersion `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeDatabase `
    -ResourceName $resourceNameDatabase `
    -LockName $lockNameDatabase `
    -LockLevel $lockLevel `
    -Force

Write-Host "List all locks on  Cosmos DB account $accountName to confirm lock creation"
Get-AzResourceLock `
    -ApiVersion $apiVersion `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeAccount `
    -ResourceName $accountName

Write-Host "Delete lock on resource $resourceNameDatabase"
Remove-AzResourceLock `
    -ApiVersion $apiVersion `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeDatabase `
    -ResourceName $resourceNameDatabase `
    -LockName $lockNameDatabase `
    -Force

Write-Host "List all locks on Cosmos DB account $accountName to confirm lock removal"
Get-AzResourceLock `
    -ApiVersion $apiVersion `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeAccount `
    -ResourceName $accountName

```

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```PowerShell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
|**Azure 资源**| |
| [New-AzResourceLock](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcelock) | 创建资源锁。 |
| [Get-AzResourceLock](https://docs.microsoft.com/powershell/module/az.resources/get-azresourcelock) | 获取资源锁，或列出资源锁。 |
| [Remove-AzResourceLock](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcelock) | 删除资源锁。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->