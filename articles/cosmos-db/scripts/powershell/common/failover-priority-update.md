---
title: 用于更改单个写入区域的 Azure Cosmos 帐户的故障转移优先级的 PowerShell 脚本
description: Azure PowerShell 脚本示例 - 对于单个写入区域的 Azure Cosmos 帐户，更改故障转移优先级或触发故障转移
ms.service: cosmos-db
ms.topic: sample
origin.date: 03/18/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: 01/20/2020
ms.author: v-yeche
ms.openlocfilehash: d637c9020771123e26295dc21a95993ba6244fdc
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328882"
---
<!--Verified successfully-->
# <a name="change-failover-priority-or-trigger-failover-for-an-azure-cosmos-account-with-single-write-region-by-using-powershell"></a>对于单个写入区域的 Azure Cosmos 帐户，使用 PowerShell 更改故障转移优先级或触发故障转移
[!INCLUDE[appliesto-all-apis](../../../includes/appliesto-all-apis.md)]

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

> [!NOTE]
> 对 `failoverPriority=0` 的区域进行的任何更改都会触发手动故障转移，并且只能对配置为手动故障转移的帐户进行更改。 对其他所有区域进行的更改只是更改了 Cosmos 帐户的故障转移优先级。
> [!NOTE]
> 此示例演示如何使用 SQL (Core) API 帐户。 若要将此示例用于其他 API，请复制相关属性，并将其应用于 API 特定的脚本

```powershell
# Reference: Az.CosmosDB | https://docs.microsoft.com/powershell/module/az.cosmosdb
# --------------------------------------------------
# Purpose
# Update Cosmos DB account: Change region failover priority.
# Note: updating location at priority 0 triggers a failover to the new location
# --------------------------------------------------
# Variables - ***** SUBSTITUTE YOUR VALUES *****
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "myaccount" # Must be all lower case
$locations = @("China North", "China East") # Regions ordered by UPDATED failover priority
# --------------------------------------------------

# Get existing Cosmos DB account
$account = Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName

# Update account failover priority
Update-AzCosmosDBAccountFailoverPriority -InputObject $account -FailoverPolicy $locations

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
| [Update-AzCosmosDBAccountFailoverPriority](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbaccountfailoverpriority) | 更新 Cosmos DB 帐户区域的故障转移优先级顺序。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

<!-- Update_Description: update meta properties, wording update, update link -->