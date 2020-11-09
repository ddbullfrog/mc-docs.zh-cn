---
title: 适用于 Azure Cosmos DB 的 Azure PowerShell 示例 - Gremlin API
description: 获取用于在 Azure Cosmos DB Gremlin API 中执行常见任务的 Azure PowerShell 示例
ms.service: cosmos-db
ms.topic: sample
origin.date: 10/13/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 28c938874055f81d82654a8522c5a9342a99addc
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328888"
---
<!--Verified successfully-->
# <a name="azure-powershell-samples-for-azure-cosmos-db-gremlin-api"></a>适用于 Azure Cosmos DB 的 Azure PowerShell 示例 - Gremlin API
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

下表包含常用于 Azure Cosmos DB 的 Azure PowerShell 脚本的链接。 使用右侧的链接可导航到 API 特定示例。 常见示例在所有 API 间是相同的。 [Azure PowerShell 参考](https://docs.microsoft.com/powershell/module/az.cosmosdb)中收录了所有 Azure Cosmos DB PowerShell cmdlet 的参考页。 请定期检查 `Az.CosmosDB` 是否有更新。 还可以从我们的 GitHub 存储库 [GitHub 上的 Cosmos DB PowerShell 示例](https://github.com/Azure/azure-docs-powershell-samples/tree/master/cosmosdb)创建这些适用于 Cosmos DB 的 PowerShell 示例的分支。

## <a name="common-samples"></a>常见示例

|任务 | 说明 |
|---|---|
|[更新帐户](scripts/powershell/common/account-update.md)| 更新 Cosmos DB 帐户的默认一致性级别。 |
|[更新帐户的区域](scripts/powershell/common/update-region.md)| 更新 Cosmos DB 帐户的区域。 |
|[更改故障转移优先级或触发故障转移](scripts/powershell/common/failover-priority-update.md)| 更改 Azure Cosmos 帐户的区域故障转移优先级或触发手动故障转移。 |
|[帐户密钥或连接字符串](scripts/powershell/common/keys-connection-strings.md)| 获取 Azure Cosmos DB 帐户的主密钥和辅助密钥、连接字符串或重新生成其帐户密钥。 |
|[创建启用 IP 防火墙的 Cosmos 帐户](scripts/powershell/common/firewall-create.md)| 创建启用 IP 防火墙的 Azure Cosmos DB 帐户。 |
|||

## <a name="gremlin-api-samples"></a>Gremlin API 示例

|任务 | 说明 |
|---|---|
|[创建帐户、数据库和图](scripts/powershell/gremlin/create.md)| 创建 Azure Cosmos 帐户、数据库和图。 |
|[创建帐户、数据库和图（具有自动缩放功能）](scripts/powershell/gremlin/autoscale.md)| 创建 Azure Cosmos 帐户、数据库和图（具有自动缩放功能）。 |
|[列出或获取数据库或图](scripts/powershell/gremlin/list-get.md)| 列出或获取数据库或图。 |
|[吞吐量操作](scripts/powershell/gremlin/throughput.md)| 数据库或图的吞吐量操作，包括获取、更新自动缩放和标准吞吐量和在其之间进行迁移。 |
|[锁定资源以防止将其删除](scripts/powershell/gremlin/lock.md)| 通过资源锁防止资源遭到删除。 |
|||

<!-- Update_Description: new article about powershell samples gremlin -->
<!--NEW.date: 11/09/2020-->