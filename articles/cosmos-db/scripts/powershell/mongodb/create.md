---
title: PowerShell 脚本用于创建 Azure Cosmos MongoDB API 数据库和集合
description: Azure PowerShell script - 创建 Azure Cosmos MongoDB API 数据库和集合
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
origin.date: 05/13/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 06/22/2020
ms.author: v-yeche
ms.openlocfilehash: 301c00d347b01d1a36cabfe94073cdf653dbce7e
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328289"
---
<!--Verified successfully-->
# <a name="create-a-database-and-collection-for-azure-cosmos-db---mongodb-api"></a>创建 Azure Cosmos DB 的数据库和集合 - MongoDB API
[!INCLUDE[appliesto-mongodb-api](../../../includes/appliesto-mongodb-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Create Cosmos MongoDB API account with automatic failover,
# a database, and a collection with dedicated throughput.
# --------------------------------------------------
Function New-RandomString{Param ([Int]$Length = 10) return $(-join ((97..122) + (48..57) | Get-Random -Count $Length | ForEach-Object {[char]$_}))}
# --------------------------------------------------
$uniqueId = New-RandomString -Length 7 # Random alphanumeric string for unique resource names
$apiKind = "MongoDB"
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "China East" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "China North" -FailoverPriority 1 -IsZoneRedundant 0

$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "cosmos-$uniqueId" # Must be all lower case
$serverVersion = "3.6" #3.2 or 3.6
$consistencyLevel = "Session"
$tags = @{Tag1 = "MyTag1"; Tag2 = "MyTag2"; Tag3 = "MyTag3"}
$databaseName = "myDatabase"
$collectionName = "myCollection"
$collectionRUs = 400
$shardKey = "user_id"
$partitionKeys = @("user_id", "user_address")
$ttlKeys = @("_ts")
$ttlInSeconds = 604800
# --------------------------------------------------
Write-Host "Creating account $accountName"
$account = New-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
    -LocationObject $locations -Name $accountName -ApiKind $apiKind -Tag $tags `
    -DefaultConsistencyLevel $consistencyLevel `
    -EnableAutomaticFailover:$true -MongoDBServerVersion $serverVersion

Write-Host "Creating database $databaseName"
$database = New-AzCosmosDBMongoDBDatabase -ParentObject $account `
    -Name $databaseName

$index1 = New-AzCosmosDBMongoDBIndex -Key $partitionKeys -Unique $true
$index2 = New-AzCosmosDBMongoDBIndex -Key $ttlKeys -TtlInSeconds $ttlInSeconds
$indexes = @($index1, $index2)

Write-Host "Creating collection $collectionName"
$collection = New-AzCosmosDBMongoDBCollection -ParentObject $database `
    -Name $collectionName -Throughput $collectionRUs `
    -Shard $shardKey -Index $indexes

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
| [New-AzCosmosDBAccount](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbaccount) | 创建 Cosmos DB 帐户。 |
| [New-AzCosmosDBMongoDBDatabase](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbmongodbdatabase) | 创建 MongoDB API 数据库。 |
| [New-AzCosmosDBMongoDBIndex](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbmongodbindex) | 创建 MongoDB API 索引。 |
| [New-AzCosmosDBMongoDBCollection](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbmongodbcollection) | 创建 MongoDB API 集合。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->