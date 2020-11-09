---
title: PowerShell 脚本：创建启用 IP 防火墙的 Azure Cosmos DB 帐户
description: Azure PowerShell 脚本示例 - 创建启用 IP 防火墙的 Azure Cosmos DB 帐户
ms.service: cosmos-db
ms.topic: sample
origin.date: 03/18/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 01/20/2020
ms.author: v-yeche
ms.openlocfilehash: 36646e119a428f2e178b6fe0d5d0536b0d619123
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328845"
---
<!--Verified successfully-->
# <a name="create-an-azure-cosmos-db-account-with-ip-firewall"></a>创建启用 IP 防火墙的 Azure Cosmos DB 帐户
[!INCLUDE[appliesto-all-apis](../../../includes/appliesto-all-apis.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

> [!NOTE]
> 此示例演示如何使用 SQL (Core) API 帐户。 若要将此示例用于其他 API，请复制相关属性，并将其应用于 API 特定的脚本

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Create Cosmos DB SQL API account with firewall
# --------------------------------------------------
Function New-RandomString{Param ([Int]$Length = 10) return $(-join ((97..122) + (48..57) | Get-Random -Count $Length | ForEach-Object {[char]$_}))}
# --------------------------------------------------
$uniqueId = New-RandomString -Length 7 # Random alphanumeric string for unique resource names
$apiKind = "Sql"
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$locations = @("China East", "China North") # Regions ordered by failover priority
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "cosmos-$uniqueId" # Must be all lower case
$consistencyLevel = "Session"
$ipFilter = @("10.0.0.0/8", "11.0.1.0/24")
$allowAzureAccess = $true # Allow access to Azure networks and portal
# --------------------------------------------------

if ($true -eq $allowAzureAccess) {
    $ipFilter += "0.0.0.0"
}

Write-Host "Creating account $accountName"
$account = New-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
    -Location $locations -Name $accountName -ApiKind $apiKind `
    -DefaultConsistencyLevel $consistencyLevel -IpRangeFilter $ipFilter `
    -EnableAutomaticFailover:$true

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
| [New-AzCosmosDBAccount](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbaccount) | 新建 Cosmos DB 帐户。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->