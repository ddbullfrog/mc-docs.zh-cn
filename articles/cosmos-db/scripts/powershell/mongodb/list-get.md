---
title: 用于列出和获取 Azure Cosmos DB 的 API for MongoDB 中操作的 PowerShell 脚本
description: Azure PowerShell 脚本 - Azure Cosmos DB 列出和获取操作 - MongoDB API
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
origin.date: 05/01/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 06/15/2020
ms.author: v-yeche
ms.openlocfilehash: 5b6ec92973384f3d705896900166abfc24ff9bde
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327577"
---
<!--Verified successfully-->
# <a name="list-and-get-databases-and-graphs-for-azure-cosmos-db---mongodb-api"></a>列出和获取 Azure Cosmos DB 的数据库和图 - MongoDB API
[!INCLUDE[appliesto-mongodb-api](../../../includes/appliesto-mongodb-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Show list and get operations for accounts, databases, and collections
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$collectionName = "myCollection"
# --------------------------------------------------

Write-Host "List all accounts in a resource group"
Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName

Write-Host "Get an account in a resource group"
Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
    -Name $accountName

Write-Host "List all databases in an account"
Get-AzCosmosDBMongoDBDatabase -ResourceGroupName $resourceGroupName `
    -AccountName $accountName

Write-Host "Get a database in an account"
Get-AzCosmosDBMongoDBDatabase -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName

Write-Host "List all collections in a database"
Get-AzCosmosDBMongoDBCollection -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName 

Write-Host "Get a collection in a database"
Get-AzCosmosDBMongoDBCollection -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName `
    -Name $collectionName

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
|**Azure Cosmos DB**| |
| [Get-AzCosmosDBAccount](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbaccount) | 列出 Cosmos DB 帐户或获取指定的 Cosmos DB 帐户。 |
| [Get-AzCosmosDBMongoDBDatabase](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbmongodbdatabase) | 列出帐户中的 MongoDB API 数据库，或在帐户中获取指定的 MogoDB API 数据库。 |
| [Get-AzCosmosDBMongoDBCollection](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbmongodbcollection) | 列出 MongoDB API 集合，或在数据库中获取指定的 MongoDB API 集合 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->