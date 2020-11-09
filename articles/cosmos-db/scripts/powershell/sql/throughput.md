---
title: 用于 Azure Cosmos DB SQL API 数据库或容器的吞吐量 (RU/s) 操作的 PowerShell 脚本
description: 用于 Azure Cosmos DB SQL API 数据库或容器的吞吐量 (RU/s) 操作的 PowerShell 脚本
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
origin.date: 10/07/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 01/20/2020
ms.author: v-yeche
ms.openlocfilehash: 67d5f865b040a15676867d1f9a0a84922c718a9d
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328642"
---
<!--Verified successfully-->
# <a name="throughput-rus-operations-with-powershell-for-a-database-or-container-for-azure-cosmos-db-core-sql-api"></a>用于 Azure Cosmos DB Core (SQL) API 的数据库或容器的使用 PowerShell 的吞吐量 (RU/s) 操作
[!INCLUDE[appliesto-sql-api](../../../includes/appliesto-sql-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="get-throughput"></a>获取吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Get database or container throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$containerName = "myContainer"
# --------------------------------------------------

Write-Host "Get database shared throughput - if none provisioned, will return error."
Get-AzCosmosDBSqlDatabaseThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName

Write-Host "Get container dedicated throughput - if none provisioned, will return error."
Get-AzCosmosDBSqlContainerThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName `
    -Name $containerName

```

## <a name="update-throughput"></a>更新吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Update container throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$containerName = "myContainer"
$newRUs = 500
# --------------------------------------------------

$throughput = Get-AzCosmosDBSqlContainerThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $containerName

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

    Update-AzCosmosDBSqlContainerThroughput -ResourceGroupName $resourceGroupName `
        -AccountName $accountName -DatabaseName $databaseName `
        -Name $containerName -Throughput $newRUs
}

```

## <a name="migrate-throughput"></a>迁移吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Migrate a database or container to autoscale or standard (manual) throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$databaseName = "myDatabase"
$containerName = "myContainer"
# --------------------------------------------------

Write-Host "Migrate database with standard throughput to autoscale throughput."
Invoke-AzCosmosDBSqlDatabaseThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName -ThroughputType Autoscale

Write-Host "Migrate database with autoscale throughput to standard throughput."
Invoke-AzCosmosDBSqlDatabaseThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $databaseName -ThroughputType Manual

Write-Host "Migrate container with standard throughput to autoscale throughput."
Invoke-AzCosmosDBSqlContainerThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $containerName -ThroughputType Autoscale

Write-Host "Migrate container with autoscale throughput to standard throughput."
Invoke-AzCosmosDBSqlContainerThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -DatabaseName $databaseName -Name $containerName -ThroughputType Manual

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
| [Get-AzCosmosDBSqlDatabaseThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbsqldatabasethroughput) | 获取在 Azure Cosmos DB Core (SQL) API 数据库上预配的吞吐量。 |
| [Get-AzCosmosDBSqlContainerThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbsqlcontainerthroughput) | 获取在 Azure Cosmos DB Core (SQL) API 容器上预配的吞吐量。 |
| [Update-AzCosmosDBSqlDatabaseThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbsqldatabase) | 更新 Azure Cosmos DB Core (SQL) API 数据库的吞吐量值。 |
| [Update-AzCosmosDBSqlContainerThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbsqlcontainer) | 更新 Azure Cosmos DB Core (SQL) API 容器的吞吐量值。 |
| [Invoke-AzCosmosDBSqlDatabaseThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbsqldatabasethroughputmigration) | 迁移 Azure Cosmos DB Core (SQL) API 数据库的吞吐量。 |
| [Invoke-AzCosmosDBSqlContainerThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbsqlcontainerthroughputmigration) | 迁移 Azure Cosmos DB Core (SQL) API 容器的吞吐量。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->