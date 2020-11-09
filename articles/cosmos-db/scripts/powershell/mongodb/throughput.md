---
title: 用于 MongoDB 的 Azure Cosmos DB API 的吞吐量 (RU/s) 操作的 PowerShell 脚本
description: 用于 MongoDB 的 Azure Cosmos DB API 的吞吐量 (RU/s) 操作的 PowerShell 脚本
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
origin.date: 10/07/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 04/27/2020
ms.author: v-yeche
ms.openlocfilehash: 12861b8c2dc3126b295d01d4178c553f87aa93d0
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327958"
---
<!--Verified successfully-->
# <a name="throughput-rus-operations-with-powershell-for-a-database-or-collection-for-azure-cosmos-db-api-for-mongodb"></a>用于 MongoDB 的 Azure Cosmos DB API 的数据库或集合的通过 PowerShell 实现的吞吐量 (RU/s) 操作
[!INCLUDE[appliesto-mongodb-api](../../../includes/appliesto-mongodb-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="get-throughput"></a>获取吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Get database or collection throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase" # Keyspace with shared throughput
$collectionName = "myCollection" # Table with dedicated throughput
# --------------------------------------------------

Write-Host "Get database shared throughput"
Get-AzCosmosDBMongoDBDatabaseThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName

Write-Host "Get collection dedicated throughput"
Get-AzCosmosDBMongoDBCollectionThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName `
    -Name $collectionName

```

## <a name="update-throughput"></a>更新吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Update collection throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$collectionName = "myCollection"
$newRUs = 500
# --------------------------------------------------

$throughput = Get-AzCosmosDBMongoDBCollectionThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $collectionName

$currentRUs = $throughput.Throughput
$minimumRUs = $throughput.MinimumThroughput

Write-Host "Current throughput is $currentRUs. Minimum allowed throughput is $minimumRUs."

if ([int]$newRUs -lt [int]$minimumRUs) {
    Write-Host "Requested new throughput of $newRUs is less than minimum allowed throughput of $minimumRUs."
    Write-Host "Using minimum allowed throughput of $minimumRUs instead."
    $newRUs = $minimumRUs
}

if ([int]$newRUs -eq [int]$currentRUs) {
    Write-Host "New throughput is the same as current throughput. No change needed."
}
else {
    Write-Host "Updating throughput to $newRUs."

    Update-AzCosmosDBMongoDBCollectionThroughput -ResourceGroupName $resourceGroupName `
        -AccountName $accountName -DatabaseName $databaseName `
        -Name $collectionName -Throughput $newRUs
}

```

## <a name="migrate-throughput"></a>迁移吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Migrate a database or collection to autoscale or standard (manual) throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$collectionName = "myCollection"
# --------------------------------------------------

Write-Host "Migrate database with standard throughput to autoscale throughput."
Invoke-AzCosmosDBMongoDBDatabaseThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName -ThroughputType Autoscale

Write-Host "Migrate database with autoscale throughput to standard throughput."
Invoke-AzCosmosDBMongoDBDatabaseThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName -ThroughputType Manual

Write-Host "Migrate collection with standard throughput to autoscale throughput."
Invoke-AzCosmosDBMongoDBCollectionThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $collectionName -ThroughputType Autoscale

Write-Host "Migrate collection with autoscale throughput to standard throughput."
Invoke-AzCosmosDBMongoDBCollectionThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $collectionName -ThroughputType Manual

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
| [Get-AzCosmosDBMongoDBDatabaseThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbmongodbdatabasethroughput) | 获取指定的 MongoDB 数据库的 Azure Cosmos DB API 的吞吐量值。 |
| [Get-AzCosmosDBMongoDBCollectionThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbmongodbcollectionthroughput) | 获取指定的 MongoDB 集合的 Azure Cosmos DB API 的吞吐量值。 |
| [Update-AzCosmosDBMongoDBDatabaseThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbmongodbdatabasethroughput) | 更新 MongoDB 数据库的 Azure Cosmos DB API 的吞吐量值。 |
| [Update-AzCosmosDBMongoDBCollectionThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbmongodbcollectionthroughput) | 更新 MongoDB 集合的 Azure Cosmos DB API 的吞吐量值。 |
| [Invoke-AzCosmosDBMongoDBDatabaseThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbmongodbdatabasethroughputmigration) | 迁移 MongoDB 集合的 Azure Cosmos DB API 的吞吐量。 |
| [Invoke-AzCosmosDBMongoDBCollectionThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbmongodbcollectionthroughputmigration) | 迁移 MongoDB 集合的 Azure Cosmos DB API 的吞吐量。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->