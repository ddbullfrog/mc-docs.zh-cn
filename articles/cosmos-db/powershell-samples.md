---
title: 适用于 Azure Cosmos DB 的 Azure PowerShell 示例
description: 获取用于在 Azure Cosmos DB 中执行常见任务的 Azure PowerShell 示例
ms.service: cosmos-db
ms.topic: sample
origin.date: 07/30/2020
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 92285286930c65a20fa9bc622f8d22756a780311
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211327"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db"></a>适用于 Azure Cosmos DB 的 Azure PowerShell 示例

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

## <a name="core-sql-api-samples"></a>Core (SQL) API 示例

|任务 | 说明 |
|---|---|
|[创建帐户、数据库和容器](scripts/powershell/sql/create.md)| 创建 Azure Cosmos DB 帐户、数据库和容器。 |
|[创建帐户、数据库和容器（具有自动缩放功能）](scripts/powershell/sql/autoscale.md)| 创建 Azure Cosmos DB 帐户、数据库和容器（具有自动缩放功能）。 |
|[创建具有大分区键的容器](scripts/powershell/sql/create-large-partition-key.md)| 创建具有大分区键的容器。 |
|[使用无索引策略创建容器](scripts/powershell/sql/create-index-none.md) | 在索引策略关闭的情况下创建 Azure Cosmos 容器。|
|[列出或获取数据库或容器](scripts/powershell/sql/list-get.md)| 列出或获取数据库或容器。 |
|[获取吞吐量](scripts/powershell/sql/throughput-get.md)| 获取数据库或容器的吞吐量。 |
|[更新吞吐量](scripts/powershell/sql/throughput-update.md)| 更新数据库或容器的吞吐量。 |
|[锁定资源以防止将其删除](scripts/powershell/sql/lock.md)| 通过资源锁防止资源遭到删除。 |
|||

## <a name="cassandra-api-samples"></a>Cassandra API 示例

|任务 | 说明 |
|---|---|
|[创建帐户、密钥空间和表](scripts/powershell/cassandra/create.md)| 创建 Azure Cosmos 帐户、密钥空间和表。 |
|[创建帐户、密钥空间和表（具有自动缩放功能）](scripts/powershell/cassandra/autoscale.md)| 创建 Azure Cosmos 帐户、密钥空间和表（具有自动缩放功能）。 |
|[列出或获取密钥空间或表](scripts/powershell/cassandra/list-get.md)| 列出或获取密钥空间或表。 |
|[获取吞吐量](scripts/powershell/cassandra/throughput-get.md)| 获取密钥空间或表的吞吐量。 |
|[更新吞吐量](scripts/powershell/cassandra/throughput-update.md)| 更新密钥空间或表的吞吐量。 |
|[锁定资源以防止将其删除](scripts/powershell/cassandra/lock.md)| 通过资源锁防止资源遭到删除。 |
|||

## <a name="mongo-db-api-samples"></a>Mongo DB API 示例

|任务 | 说明 |
|---|---|
|[创建帐户、数据库和集合](scripts/powershell/mongodb/create.md)| 创建 Azure Cosmos 帐户、数据库和集合。 |
|[创建帐户、数据库和集合（具有自动缩放功能）](scripts/powershell/mongodb/autoscale.md)| 创建 Azure Cosmos 帐户、数据库和集合（具有自动缩放功能）。 |
|[列出或获取数据库或集合](scripts/powershell/mongodb/list-get.md)| 列出或获取数据库或集合。 |
|[获取吞吐量](scripts/powershell/mongodb/throughput-get.md)| 获取数据库或集合的吞吐量。 |
|[更新吞吐量](scripts/powershell/mongodb/throughput-update.md)| 更新数据库或集合的吞吐量。 |
|[锁定资源以防止将其删除](scripts/powershell/mongodb/lock.md)| 通过资源锁防止资源遭到删除。 |
|||

## <a name="gremlin-api-samples"></a>Gremlin API 示例

|任务 | 说明 |
|---|---|
|[创建帐户、数据库和图](scripts/powershell/gremlin/create.md)| 创建 Azure Cosmos 帐户、数据库和图。 |
|[创建帐户、数据库和图（具有自动缩放功能）](scripts/powershell/gremlin/autoscale.md)| 创建 Azure Cosmos 帐户、数据库和图（具有自动缩放功能）。 |
|[列出或获取数据库或图](scripts/powershell/gremlin/list-get.md)| 列出或获取数据库或图。 |
|[获取吞吐量](scripts/powershell/gremlin/throughput-get.md)| 获取数据库或图的吞吐量。 |
|[更新吞吐量](scripts/powershell/gremlin/throughput-update.md)| 更新数据库或图的吞吐量。 |
|[锁定资源以防止将其删除](scripts/powershell/gremlin/lock.md)| 通过资源锁防止资源遭到删除。 |
|||

## <a name="table-api-samples"></a>表 API 示例

|任务 | 说明 |
|---|---|
|[创建帐户和表](scripts/powershell/table/create.md)| 创建 Azure Cosmos 帐户和表。 |
|[创建帐户和表（具有自动缩放功能）](scripts/powershell/table/autoscale.md)| 创建 Azure Cosmos 帐户和表（具有自动缩放功能）。 |
|[列出或获取表](scripts/powershell/table/list-get.md)| 列出或获取表。 |
|[获取吞吐量](scripts/powershell/table/throughput-get.md)| 获取表的吞吐量。 |
|[更新吞吐量](scripts/powershell/table/throughput-update.md)| 更新表的吞吐量。 |
|[锁定资源以防止将其删除](scripts/powershell/table/lock.md)| 通过资源锁防止资源遭到删除。 |
|||

<!-- Update_Description: new article about powershell samples -->
<!--NEW.date: 10/05/2020-->