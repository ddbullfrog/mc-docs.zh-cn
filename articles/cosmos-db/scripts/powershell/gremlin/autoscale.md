---
title: 用于创建 Azure Cosmos DB Gremlin API 数据库和图（具有自动缩放功能）的 PowerShell 脚本
description: Azure PowerShell 脚本 - Azure Cosmos DB 创建 Gremlin API 数据库和图（具有自动缩放功能）
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
origin.date: 07/30/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 08/17/2020
ms.author: v-yeche
ms.openlocfilehash: f2cc174f0efd46677044ab315ac70b1684340463
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328819"
---
<!--Verified successfully-->
# <a name="create-a-database-and-graph-with-autoscale-for-azure-cosmos-db---gremlin-api"></a>为 Azure Cosmos DB 创建数据库和图（具有自动缩放功能）- Gremlin API
[!INCLUDE[appliesto-gremlin-api](../../../includes/appliesto-gremlin-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Create Cosmos Gremlin API account, database, and graph
# with autoscale throughput
# --------------------------------------------------
Function New-RandomString{Param ([Int]$Length = 10) return $(-join ((97..122) + (48..57) | Get-Random -Count $Length | ForEach-Object {[char]$_}))}
# --------------------------------------------------
$uniqueId = New-RandomString -Length 7 # Random alphanumeric string for unique resource names
$apiKind = "Gremlin"
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "China East" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "China North" -FailoverPriority 1 -IsZoneRedundant 0

$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "cosmos-$uniqueId" # Must be all lower case
$consistencyLevel = "Session"
$databaseName = "myDatabase"
$graphName = "myGraph"
$autoscaleMaxThroughput = 4000 #minimum = 4000
$partitionKeys = @("/myPartitionKey")
# --------------------------------------------------

Write-Host "Creating account $accountName"
$account = New-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
    -LocationObject $locations -Name $accountName -ApiKind $apiKind `
    -DefaultConsistencyLevel $consistencyLevel `
    -EnableAutomaticFailover:$true

Write-Host "Creating database $databaseName"
$database = New-AzCosmosDBGremlinDatabase -ParentObject $account `
    -Name $databaseName

Write-Host "Creating graph $graphName"
$graph = New-AzCosmosDBGremlinGraph -ParentObject $database `
    -Name $graphName -AutoscaleMaxThroughput $autoscaleMaxThroughput `
    -PartitionKeyKind Hash -PartitionKeyPath $partitionKeys

```

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 注释 |
|---|---|
|**Azure Cosmos DB**| |
| [New-AzCosmosDBAccount](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbaccount) | 创建 Cosmos DB 帐户。 |
| [New-AzCosmosDBGremlinDatabase](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbgremlindatabase) | 创建 Gremlin API 数据库。 |
| [New-AzCosmosDBGremlinGraph](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbgremlingraph) | 创建 Gremlin API 图。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->