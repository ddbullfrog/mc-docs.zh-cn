---
title: cosmosdb_sql_request 插件 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 cosmosdb_sql_request 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: miwalia
ms.service: data-explorer
ms.topic: reference
origin.date: 09/11/2020
ms.date: 10/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 6a2ec00eef976914957e5727e7a996ade5ba52ac
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106634"
---
# <a name="cosmosdb_sql_request-plugin"></a>cosmosdb_sql_request 插件

::: zone pivot="azuredataexplorer"

`cosmosdb_sql_request` 插件会将 SQL 查询发送到 Cosmos DB SQL 网络终结点，并返回查询的结果。 此插件主要为查询小数据集而设计，例如，使用存储在 [Azure Cosmos DB](/azure/cosmos-db/) 中的参考数据来扩充数据。

## <a name="syntax"></a>语法

`evaluate` `cosmosdb_sql_request` `(` *ConnectionString* `,` *SqlQuery* [`,` *SqlParameters* [`,` *Options* ]] `)`

## <a name="arguments"></a>参数

|参数名称 | 说明 | 必需/可选 | 
|---|---|---|
| *ConnectionString* | 一个表示连接字符串的 `string` 文本，该连接字符串指向要查询的 Cosmos DB 集合。 它必须包括“AccountEndpoint”、“Database”和“Collection”。 如果主密钥用于身份验证，则可能包括“AccountKey”。 <br> **示例：** `'AccountEndpoint=https://cosmosdbacc.documents.azure.com:443/ ;Database=MyDatabase;Collection=MyCollection;AccountKey=' h'R8PM...;'`| 必须 |
| *SqlQuery*| 一个 `string` 文本，指示要执行的查询。 | 必须 |
| SqlParameters | `dynamic` 类型的常数值，用于保存作为参数随查询传递的键值对。 参数名称必须以 `@` 开头。 | 可选 |
| *选项* | `dynamic` 类型的常数值，它将更高级的设置保存为键值对。 | 可选 |
|| ----支持的选项设置包括：-----
|      `armResourceId` | 从 Azure 资源管理器检索 API 密钥 <br> **示例：** `/subscriptions/a0cd6542-7eaf-43d2-bbdd-b678a869aad1/resourceGroups/ cosmoddbresourcegrouput/providers/Microsoft.DocumentDb/databaseAccounts/cosmosdbacc`| 
|  `token` | 提供用于通过 Azure 资源管理器进行身份验证的 Azure AD 访问令牌。
| `preferredLocations` | 控制从哪个区域查询数据。 <br> **示例：** `['China East2']` | |  

## <a name="set-callout-policy"></a>设置标注策略

该插件会对 Cosmos DB 进行标注。 请确保群集的[标注策略](../management/calloutpolicy.md)允许对目标 CosmosDbUri 进行 `cosmosdb` 类型的调用。

以下示例演示如何为 Cosmos DB 定义标注策略。 建议将其限制为特定的终结点（`my_endpoint1`、`my_endpoint2`）。

```kusto
[
  {
    "CalloutType": "CosmosDB",
    "CalloutUriRegex": "my_endpoint1.documents.azure.com",
    "CanCall": true
  },
  {
    "CalloutType": "CosmosDB",
    "CalloutUriRegex": "my_endpoint2.documents.azure.com",
    "CanCall": true
  }
]
```

以下示例显示了针对 `cosmosdb` CalloutType 的 alter callout policy 命令

```kusto
.alter cluster policy callout @'[{"CalloutType": "cosmosdb", "CalloutUriRegex": "\\.documents\\.azure\\.com", "CanCall": true}]'
```

## <a name="examples"></a>示例

### <a name="query-cosmos-db"></a>查询 Cosmos DB

以下示例使用 cosmosdb_sql_request 插件发送 SQL 查询，以便使用其 SQL API 从 Cosmos DB 提取数据。

```kusto
evaluate cosmosdb_sql_request(
  'AccountEndpoint=https://cosmosdbacc.documents.azure.com:443/;Database=MyDatabase;Collection=MyCollection;AccountKey=' h'R8PM...;',
  'SELECT * from c')
```

### <a name="query-cosmos-db-with-parameters"></a>使用参数查询 Cosmos DB

以下示例使用 SQL 查询参数从备用区域查询数据。 有关详细信息，请参阅 [`preferredLocations`](/azure/cosmos-db/tutorial-global-distribution-sql-api?tabs=dotnetv2%2Capi-async#preferred-locations)。

```kusto
evaluate cosmosdb_sql_request(
    'AccountEndpoint=https://cosmosdbacc.documents.azure.com:443/;Database=MyDatabase;Collection=MyCollection;AccountKey=' h'R8PM...;',
    "SELECT c.id, c.lastName, @param0 as Column0 FROM c WHERE c.dob >= '1970-01-01T00:00:00Z'",
    dynamic({'@param0': datetime(2019-04-16 16:47:26.7423305)}),
    dynamic({'preferredLocations': ['China East2']}))
| where lastName == 'Smith'
```

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
