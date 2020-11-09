---
title: 用于 Azure Cosmos DB 的 Azure CLI 示例
description: 用于 Azure Cosmos DB 的 Azure CLI 示例
author: rockboyfor
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
origin.date: 07/29/2020
ms.date: 08/17/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: ca9a6c8a58020ade571a2f9e3ab258ed2c242b83
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328321"
---
# <a name="azure-cli-samples-for-azure-cosmos-db"></a>用于 Azure Cosmos DB 的 Azure CLI 示例

下表包括用于 Azure Cosmos DB 的示例 Azure CLI 脚本的链接。 使用右侧的链接可导航到 API 特定示例。 常见示例在所有 API 间是相同的。 [Azure CLI 参考](https://docs.azure.cn/cli/cosmosdb?view=azure-cli-latest)中收录了所有 Azure Cosmos DB CLI 命令的参考页。 还可以在 [Azure Cosmos DB CLI GitHub 存储库](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb)中找到 Azure Cosmos DB CLI 脚本示例。

这些示例要求使用 Azure CLI 2.9.1 版或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)

## <a name="common-samples"></a>常见示例

这些示例适用于所有 Azure Cosmos DB API

|任务 | 说明 |
|---|---|
| [添加或故障转移区域](scripts/cli/common/regions.md?toc=%2fcli%2ftoc.json) | 添加区域、更改故障转移优先级、触发手动故障转移。|
| [帐户密钥和连接字符串](scripts/cli/common/keys.md?toc=%2fcli%2ftoc.json) | 列出帐户密钥、只读密钥，重新生成密钥并列出连接字符串。|
| [使用 IP 防火墙保护](scripts/cli/common/ipfirewall.md?toc=%2fcli%2ftoc.json)| 创建配置了 IP 防火墙的 Cosmos 帐户。|
| [使用服务终结点保护新帐户](scripts/cli/common/service-endpoints.md?toc=%2fcli%2ftoc.json)| 使用服务终结点创建 Cosmos 帐户并确保其安全。|
| [使用服务终结点保护现有帐户](scripts/cli/common/service-endpoints-ignore-missing-vnet.md?toc=%2fcli%2ftoc.json)| 最终配置子网后，更新 Cosmos 帐户以使用服务终结点进行保护。|
|||

## <a name="core-sql-api-samples"></a>Core (SQL) API 示例

|任务 | 说明 |
|---|---|
| [创建 Azure Cosmos 帐户、数据库和容器](scripts/cli/sql/create.md?toc=%2fcli%2ftoc.json)| 为 Core (SQL) API 创建 Azure Cosmos DB 帐户、数据库和容器。 |
| [创建 Azure Cosmos 帐户、数据库和容器（具有自动缩放功能）](scripts/cli/sql/autoscale.md?toc=%2fcli%2ftoc.json)| 为 Core (SQL) API 创建 Azure Cosmos DB 帐户、数据库和容器（具有自动缩放功能）。 |
| [更改吞吐量](scripts/cli/sql/throughput.md?toc=%2fcli%2ftoc.json) | 更新数据库和容器的 RU/秒。|
| [锁定资源以防止将其删除](scripts/cli/sql/lock.md?toc=%2fcli%2ftoc.json)| 使用资源锁防止删除资源。|
|||

## <a name="mongodb-api-samples"></a>MongoDB API 示例

|任务 | 说明 |
|---|---|
| [创建 Azure Cosmos 帐户、数据库和集合](scripts/cli/mongodb/create.md?toc=%2fcli%2ftoc.json)| 为 MongoDB API 创建 Azure Cosmos DB 帐户、数据库和集合。 |
| [创建 Azure Cosmos 帐户、数据库（具有自动缩放功能）以及两个集合（具有共享吞吐量）](scripts/cli/mongodb/autoscale.md?toc=%2fcli%2ftoc.json)| 为 MongoDB API 创建 Azure Cosmos DB 帐户、数据库（具有自动缩放功能）以及两个集合（具有共享吞吐量）。 |
| [更改吞吐量](scripts/cli/mongodb/throughput.md?toc=%2fcli%2ftoc.json) | 更新数据库和集合的 RU/秒。|
| [锁定资源以防止将其删除](scripts/cli/mongodb/lock.md?toc=%2fcli%2ftoc.json)| 使用资源锁防止删除资源。|
|||

## <a name="cassandra-api-samples"></a>Cassandra API 示例

|任务 | 说明 |
|---|---|
| [创建 Azure Cosmos 帐户、密钥空间和表](scripts/cli/cassandra/create.md?toc=%2fcli%2ftoc.json)| 为 Cassandra API 创建 Azure Cosmos DB 帐户、密钥空间和表。 |
| [创建 Azure Cosmos 帐户、密钥空间和表（具有自动缩放功能）](scripts/cli/cassandra/autoscale.md?toc=%2fcli%2ftoc.json)| 为 Cassandra API 创建 Azure Cosmos DB 帐户、密钥空间和表（具有自动缩放功能）。 |
| [更改吞吐量](scripts/cli/cassandra/throughput.md?toc=%2fcli%2ftoc.json) | 更新密钥空间和表的 RU/秒。|
| [锁定资源以防止将其删除](scripts/cli/cassandra/lock.md?toc=%2fcli%2ftoc.json)| 使用资源锁防止删除资源。|
|||

## <a name="gremlin-api-samples"></a>Gremlin API 示例

|任务 | 说明 |
|---|---|
| [创建 Azure Cosmos 帐户、数据库和图](scripts/cli/gremlin/create.md?toc=%2fcli%2ftoc.json)| 为 Gremlin API 创建 Azure Cosmos DB 帐户、数据库和图。 |
| [创建 Azure Cosmos 帐户、数据库和图（具有自动缩放功能）](scripts/cli/gremlin/autoscale.md?toc=%2fcli%2ftoc.json)| 为 Gremlin API 创建 Azure Cosmos DB 帐户、数据库和图（具有自动缩放功能）。 |
| [更改吞吐量](scripts/cli/gremlin/throughput.md?toc=%2fcli%2ftoc.json) | 更新数据库和图的 RU/秒。|
| [锁定资源以防止将其删除](scripts/cli/gremlin/lock.md?toc=%2fcli%2ftoc.json)| 使用资源锁防止删除资源。|
|||

## <a name="table-api-samples"></a>表 API 示例

|任务 | 说明 |
|---|---|
| [创建 Azure Cosmos 帐户和表](scripts/cli/table/create.md?toc=%2fcli%2ftoc.json)| 为表 API 创建 Azure Cosmos DB 帐户和表。 |
| [创建 Azure Cosmos 帐户和表（具有自动缩放功能）](scripts/cli/table/autoscale.md?toc=%2fcli%2ftoc.json)| 为表 API 创建 Azure Cosmos DB 帐户和表（具有自动缩放功能）。 |
| [更改吞吐量](scripts/cli/table/throughput.md?toc=%2fcli%2ftoc.json) | 更新表的 RU/秒。|
| [锁定资源以防止将其删除](scripts/cli/table/lock.md?toc=%2fcli%2ftoc.json)| 通过资源锁防止资源遭到删除。|
|||

<!-- Update_Description: update meta properties, wording update, update link -->