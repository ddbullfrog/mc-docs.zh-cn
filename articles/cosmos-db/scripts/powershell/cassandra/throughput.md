---
title: 用于 Azure Cosmos DB Cassandra API 资源的吞吐量 (RU/s) 操作的 PowerShell 脚本
description: 用于 Azure Cosmos DB Cassandra API 资源的吞吐量 (RU/s) 操作的 PowerShell 脚本
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: sample
origin.date: 10/07/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 06/15/2020
ms.author: v-yeche
ms.openlocfilehash: b0d3e5d5f9ed6ff4c4edb427707f257397563d8b
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328643"
---
<!--Verified successfully-->
# <a name="throughput-rus-operations-with-powershell-for-a-keyspace-or-table-for-azure-cosmos-db---cassandra-api"></a>使用 PowerShell 实现的用于 Azure Cosmos DB - Cassandra API 的密钥空间或表的吞吐量 (RU/s) 操作
[!INCLUDE[appliesto-cassandra-api](../../../includes/appliesto-cassandra-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="get-throughput"></a>获取吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Get keyspace or table throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$keyspaceName = "mykeyspace" # Keyspace with shared throughput
$tableName = "mytable" # Table with dedicated throughput
# --------------------------------------------------

Write-Host "Get keyspace shared throughput"
Get-AzCosmosDBCassandraKeyspaceThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $keyspaceName

Write-Host "Get table dedicated throughput"
Get-AzCosmosDBCassandraTableThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -KeyspaceName $keyspaceName `
    -Name $tableName

```

## <a name="update-throughput"></a>更新吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Update table throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$keyspaceName = "mykeyspace"
$tableName = "mytable"
$newRUs = 500
# --------------------------------------------------
$throughput = Get-AzCosmosDBCassandraTableThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -KeyspaceName $keyspaceName -Name $tableName

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

    Update-AzCosmosDBCassandraTableThroughput -ResourceGroupName $resourceGroupName `
        -AccountName $accountName -KeyspaceName $keyspaceName `
        -Name $tableName -Throughput $newRUs
}

```

## <a name="migrate-throughput"></a>迁移吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Migrate a keyspace or table to autoscale or standard (manual) throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$keyspaceName = "myKeyspace"
$tableName = "myTable"
# --------------------------------------------------

Write-Host "Migrate keyspace with standard throughput to autoscale throughput."
Invoke-AzCosmosDBCassandraKeyspaceThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $keyspaceName -ThroughputType Autoscale

Write-Host "Migrate keyspace with autoscale throughput to standard throughput."

Invoke-AzCosmosDBCassandraKeyspaceThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $keyspaceName -ThroughputType Manual

Write-Host "Migrate table with standard throughput to autoscale throughput."
Invoke-AzCosmosDBCassandraTableThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -KeyspaceName $keyspaceName -Name $tableName -ThroughputType Autoscale

Write-Host "Migrate table with autoscale throughput to standard throughput."
Invoke-AzCosmosDBCassandraTableThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -KeyspaceName $keyspaceName -Name $tableName -ThroughputType Manual

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
| [Get-AzCosmosDBCassandraKeyspaceThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbcassandrakeyspacethroughput) | 获取 Cassandra API 密钥空间的吞吐量值。 |
| [Get-AzCosmosDBCassandraTableThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbcassandratablethroughput) | 获取 Cassandra API 表的吞吐量值。 |
| [Update-AzCosmosDBCassandraKeyspaceThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbcassandrakeyspacethroughput) | 更新 Cassandra API 密钥空间的吞吐量值。 |
| [Update-AzCosmosDBCassandraTableThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbcassandratablethroughput) | 更新 Cassandra API 表的吞吐量值。 |
| [Invoke-AzCosmosDBCassandraKeyspaceThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbcassandrakeyspacethroughputmigration) | 迁移 Cassandra API 密钥空间的吞吐量。 |
| [Invoke-AzCosmosDBCassandraTableThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbcassandratablethroughputmigration) | 迁移 Cassandra API 表的吞吐量。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->
