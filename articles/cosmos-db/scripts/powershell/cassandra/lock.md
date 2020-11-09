---
title: PowerShell 脚本创建用于 Azure Cosmos Cassandra API 密钥空间和表的资源锁
description: 创建用于 Azure Cosmos Cassandra API 密钥空间和表的资源锁
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: sample
origin.date: 06/12/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 08/03/2020
ms.author: v-yeche
ms.openlocfilehash: 670b97d4af621d07d1952178d753e51a90ee613b
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327454"
---
<!--Verified successfully-->
# <a name="create-a-resource-lock-for-azure-cosmos-cassandra-api-keyspace-and-table-using-azure-powershell"></a>使用 Azure PowerShell 创建用于 Azure Cosmos Cassandra API 密钥空间和表的资源锁
[!INCLUDE[appliesto-cassandra-api](../../../includes/appliesto-cassandra-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

> [!IMPORTANT]
> 对于使用任何 Cassandra SDK、CQL Shell 或 Azure 门户进行连接的用户所做的更改，除非先在启用了 `disableKeyBasedMetadataWriteAccess` 属性的情况下锁定 Cosmos DB 帐户，否则资源锁无效。 要详细了解如何启用此属性，请参阅[防止 SDK 更改](../../../role-based-access-control.md#prevent-sdk-changes)。

## <a name="sample-script"></a>示例脚本

```powershell
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
$databaseName = "mykeyspace"
$collectionName = "mytable"
# *****
$apiVersion = "2020-04-01" # Cosmos DB RP API version
$lockLevel = "CanNotDelete" # CanNotDelete or ReadOnly

$resourceTypeAccount = "Microsoft.DocumentDB/databaseAccounts"
$resourceTypeDatabase = "$resourceTypeAccount/cassandraKeyspaces"
$resourceTypeCollection = "$resourceTypeDatabase/tables"

$resourceNameDatabase = "$accountName/$databaseName"
$lockNameDatabase = "$accountName-$databaseName-Lock"

$resourceNameCollection = "$accountName/$databaseName/$collectionName"
$lockNameCollection = "$accountName-$databaseName-$collectionName-Lock"
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

Write-Host "Create a $lockLevel lock on resource $resourceNameCollection"
New-AzResourceLock `
    -ApiVersion $apiVersion `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeCollection `
    -ResourceName $resourceNameCollection `
    -LockName $lockNameCollection `
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

Write-Host "Delete lock on resource $resourceNameCollection"
Remove-AzResourceLock `
    -ApiVersion $apiVersion `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeCollection `
    -ResourceName $resourceNameCollection `
    -LockName $lockNameCollection `
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

```powershell
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