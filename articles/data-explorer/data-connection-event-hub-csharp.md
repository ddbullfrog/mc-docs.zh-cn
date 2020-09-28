---
title: 使用 C# 为 Azure 数据资源管理器创建事件中心数据连接
description: 本文介绍如何使用 C# 为 Azure 数据资源管理器创建事件中心数据连接。
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 10/07/2019
ms.date: 09/24/2020
ms.openlocfilehash: 5bd8e140356f7bba958eea5a7aa92d8403cc766a
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146578"
---
# <a name="create-an-event-hub-data-connection-for-azure-data-explorer-by-using-c"></a>使用 C# 为 Azure 数据资源管理器创建事件中心数据连接

> [!div class="op_single_selector"]
> * [门户](ingest-data-event-hub.md)
> * [C#](data-connection-event-hub-csharp.md)
> * [Python](data-connection-event-hub-python.md)
> * [Azure Resource Manager 模板](data-connection-event-hub-resource-manager.md)

[!INCLUDE [data-connector-intro](includes/data-connector-intro.md)]
在本文中，你将使用 C# 为 Azure 数据资源管理器创建事件中心数据连接。

## <a name="prerequisites"></a>先决条件

* 如果尚未安装 Visual Studio 2019，可以下载并使用**免费的** [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。
* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* 创建[群集和数据库](create-cluster-database-csharp.md)
* 创建[表和列映射](net-standard-ingest-data.md#create-a-table-on-your-test-cluster)
* 设置[数据库和表策略](database-table-policies-csharp.md)（可选）
* 创建[包含要引入的数据的事件中心](ingest-data-event-hub.md#create-an-event-hub)。 

[!INCLUDE [data-explorer-data-connection-install-nuget-csharp](includes/data-explorer-data-connection-install-nuget-csharp.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-event-hub-data-connection"></a>添加事件中心数据连接

以下示例演示如何以编程方式添加事件中心数据连接。 请参阅[连接到事件中心](ingest-data-event-hub.md#connect-to-the-event-hub)，了解如何使用 Azure 门户添加事件中心数据连接。

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
var authenticationContext = new AuthenticationContext($"https://login.chinacloudapi.cn/{tenantId}");
var credential = new ClientCredential(clientId, clientSecret);
var result = await authenticationContext.AcquireTokenAsync(resource: "https://management.core.chinacloudapi.cn/", clientCredential: credential);

var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);

var kustoManagementClient = new KustoManagementClient(credentials)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the Prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
var dataConnectionName = "myeventhubconnect";
//The event hub that is created as part of the Prerequisites
var eventHubResourceId = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.EventHub/namespaces/xxxxxx/eventhubs/xxxxxx";
var consumerGroup = "$Default";
var location = "China East 2";
//The table and column mapping are created as part of the Prerequisites
var tableName = "StormEvents";
var mappingRuleName = "StormEvents_CSV_Mapping";
var dataFormat = DataFormat.CSV;
await kustoManagementClient.DataConnections.CreateOrUpdateAsync(resourceGroupName, clusterName, databaseName, dataConnectionName, 
    new EventHubDataConnection(eventHubResourceId, consumerGroup, location: location, tableName: tableName, mappingRuleName: mappingRuleName, dataFormat: dataFormat));
```

|**设置** | **建议的值** | **字段说明**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 租户 ID。 也称为目录 ID。|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 用于创建资源的订阅 ID。|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 可以访问租户中资源的应用程序的客户端 ID。|
| clientSecret | *xxxxxxxxxxxxxx* | 可以访问租户中资源的应用程序的客户端密码。|
| resourceGroupName | *testrg* | 包含群集的资源组的名称。|
| clusterName | mykustocluster** | 群集的名称。|
| databaseName | mykustodatabase** | 群集中目标数据库的名称。|
| dataConnectionName | *myeventhubconnect* | 所需的数据连接名称。|
| tableName | *StormEvents* | 目标数据库中目标表的名称。|
| mappingRuleName | *StormEvents_CSV_Mapping* | 与目标表相关的列映射的名称。|
| dataFormat | *csv* | 消息的数据格式。|
| eventHubResourceId | 资源 ID | 包含要引入的数据的事件中心的资源 ID。 |
| consumerGroup | *$Default* | 事件中心的使用者组。|
| location | *中国东部 2* | 数据连接资源的位置。|

## <a name="generate-data"></a>生成数据

请参阅可生成数据并将其发送到事件中心的[示例应用](https://github.com/Azure-Samples/event-hubs-dotnet-ingest)。

事件可以包含一个或多个记录（直到达到事件的大小限制）。 在下面的示例中，我们将发送两个事件，每个事件有五个追加的记录：

```csharp
var events = new List<EventData>();
var data = string.Empty;
var recordsPerEvent = 5;
var rand = new Random();
var counter = 0;

for (var i = 0; i < 10; i++)
{
    // Create the data
    var metric = new Metric { Timestamp = DateTime.UtcNow, MetricName = "Temperature", Value = rand.Next(-30, 50) }; 
    var data += JsonConvert.SerializeObject(metric) + Environment.NewLine;
    counter++;

    // Create the event
    if (counter == recordsPerEvent)
    {
        var eventData = new EventData(Encoding.UTF8.GetBytes(data));
        events.Add(eventData);

        counter = 0;
        data = string.Empty;
    }
}

// Send events
eventHubClient.SendAsync(events).Wait();
```

[!INCLUDE [data-explorer-data-connection-clean-resources-csharp](includes/data-explorer-data-connection-clean-resources-csharp.md)]
