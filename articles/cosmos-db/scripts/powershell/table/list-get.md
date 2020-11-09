---
title: 用于列出和获取 Azure Cosmos DB 表 API 操作的 PowerShell 脚本
description: Azure PowerShell 脚本 - Azure Cosmos DB 列出和获取操作 - 表 API
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
origin.date: 07/31/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 06/15/2020
ms.author: v-yeche
ms.openlocfilehash: dc27c40696ee7126caddf2a1fee1a1084cd284f1
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328658"
---
<!--Verified successfully-->
# <a name="list-and-get-tables-for-azure-cosmos-db---table-api"></a>列出和获取 Azure Cosmos DB 的表 - 表 API
[!INCLUDE[appliesto-table-api](../../../includes/appliesto-table-api.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Show list and get operations for Azure Cosmos DB Table API
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$tableName = "myTable"
# --------------------------------------------------

Write-Host "List all accounts in a resource group"
Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName

Write-Host "Get an account in a resource group"
Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
    -Name $accountName

Write-Host "List all tables in an account"
Get-AzCosmosDBTable -ResourceGroupName $resourceGroupName `
    -AccountName $accountName

Write-Host "Get a table in an account including throughput"
Get-AzCosmosDBTable -ResourceGroupName $resourceGroupName `
    -AccountName $accountName -Name $tableName

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
| [Get-AzCosmosDBTable](https://docs.microsoft.com/powershell/module/az.cosmosdb/get-azcosmosdbtable) | 列出帐户中的表 API 表，或在帐户中获取指定的表 API 表。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->