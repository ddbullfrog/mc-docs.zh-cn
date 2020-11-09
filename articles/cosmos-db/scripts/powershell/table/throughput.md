---
title: 用于 Azure Cosmos DB 表 API 的吞吐量 (RU/s) 操作的 PowerShell 脚本
description: 用于 Azure Cosmos DB 表 API 的吞吐量 (RU/s) 操作的 PowerShell 脚本
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
origin.date: 10/07/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 01/20/2020
ms.author: v-yeche
ms.openlocfilehash: 8ec4e5d2f403acc6546ee80df16315e2bcc131ab
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328828"
---
<!--Verified successfully-->
# <a name="throughput-rus-operations-with-powershell-for-a-table-for-azure-cosmos-db---table-api"></a>针对用于 Azure Cosmos DB 表 API 的表，使用 PowerShell 实现的吞吐量 (RU/s) 操作
[!INCLUDE[appliesto-table-api](../../../includes/appliesto-table-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="get-throughput"></a>获取吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Get RU/s throughput for Azure Cosmos DB Table API table
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$tableName = "myTable"
# --------------------------------------------------

Get-AzCosmosDBTableThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $tableName

```

## <a name="update-throughput"></a>更新吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Update Table throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$tableName = "myTable"
$newRUs = 500
# --------------------------------------------------

$throughput = Get-AzCosmosDBTableThroughput -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $tableName

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

    Update-AzCosmosDBTableThroughput -ResourceGroupName $resourceGroupName `
        -AccountName $accountName -Name $tableName `
        -Throughput $newRUs
}

```

## <a name="migrate-throughput"></a>迁移吞吐量

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Migrate a table to autoscale or standard (manual) throughput
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$tableName = "myTable"
# --------------------------------------------------

Write-Host "Migrate table with standard throughput to autoscale throughput."
Invoke-AzCosmosDBTableThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $tableName -ThroughputType Autoscale

Write-Host "Migrate table with autoscale throughput to standard throughput."
Invoke-AzCosmosDBTableThroughputMigration -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $tableName -ThroughputType Manual

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
| [Get-AzCosmosDBTableThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbtablethroughput) | 获取指定表 API 表的吞吐量值。 |
| [Update-AzCosmosDBMongoDBCollectionThroughput](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbsqldatabasethroughput) | 更新表 API 表的吞吐量值。 |
| [Invoke-AzCosmosDBTableThroughputMigration](https://docs.microsoft.com/powershell/module/az.cosmosdb/invoke-azcosmosdbtablethroughputmigration) | 迁移表 API 表的吞吐量。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->