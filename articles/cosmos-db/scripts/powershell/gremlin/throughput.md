---
title: 用于 Azure Cosmos DB Gremlin API 的吞吐量 (RU/s) 操作的 PowerShell 脚本
description: 用于 Gremlin API 的吞吐量 (RU/s) 操作的 PowerShell 脚本
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
origin.date: 10/07/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 06/15/2020
ms.author: v-yeche
ms.openlocfilehash: 718c6fda125f6b198705f355cae6af106196b7c7
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327576"
---
<!--Verified successfully from Rename articles-->
# <a name="throughput-rus-operations-with-powershell-for-a-database-or-graph-for-azure-cosmos-db---gremlin-api"></a>用于 Azure Cosmos DB - Gremlin API 的数据库或图的使用 PowerShell 实现的吞吐量 (RU/s) 操作
[!INCLUDE[appliesto-gremlin-api](../../../includes/appliesto-gremlin-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="get-throughput"></a>获取吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Get database or graph throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$graphName = "myGraph"
# --------------------------------------------------

Write-Host "Get database shared throughput"
Get-AzCosmosDBGremlinDatabaseThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName

Write-Host "Get graph dedicated throughput"
Get-AzCosmosDBGremlinGraphThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName `
    -Name $graphName

```

## <a name="update-throughput"></a>更新吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Update graph throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$graphName = "myGraph"
$newRUs = 500
# --------------------------------------------------

$throughput = Get-AzCosmosDBGremlinGraphThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName `
    -Name $graphName

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

    Update-AzCosmosDBGremlinGraphThroughput -ResourceGroupName $resourceGroupName `
        -AccountName $accountName -DatabaseName $databaseName `
        -Name $graphName -Throughput $newRUs
}

```

## <a name="migrate-throughput"></a>迁移吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Migrate a database or graph to autoscale or standard (manual) throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$graphName = "myGraph"
# --------------------------------------------------

Write-Host "Migrate database with standard throughput to autoscale throughput."
Invoke-AzCosmosDBGremlinDatabaseThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName -ThroughputType Autoscale

Write-Host "Migrate database with autoscale throughput to standard throughput."
Invoke-AzCosmosDBGremlinDatabaseThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName -ThroughputType Manual

Write-Host "Migrate graph with standard throughput to autoscale throughput."
Invoke-AzCosmosDBGremlinGraphThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $graphName -ThroughputType Autoscale

Write-Host "Migrate graph with autoscale throughput to standard throughput."
Invoke-AzCosmosDBGremlinGraphThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $graphName -ThroughputType Manual

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
| [Get-AzCosmosDBGremlinDatabaseThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbgremlindatabasethroughput) | 获取 Gremlin API 数据库的吞吐量值。 |
| [Get-AzCosmosDBGremlinGraphThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbgremlingraphthroughput) | 获取 Gremlin API 图的吞吐量值。 |
| [Update-AzCosmosDBGremlinDatabaseThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbgremlindatabasethroughput) | 更新 Gremlin API 数据库的吞吐量值。 |
| [Update-AzCosmosDBGremlinGraphThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbgremlingraphthroughput) | 更新 Gremlin API 图的吞吐量值。 |
| [Invoke-AzCosmosDBGremlinDatabaseThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbgremlindatabasethroughputmigration) | 迁移 Gremlin API 数据库的吞吐量。 |
| [Invoke-AzCosmosDBGremlinGraphThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbgremlingraphthroughputmigration) | 迁移 Gremlin API 图的吞吐量。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->