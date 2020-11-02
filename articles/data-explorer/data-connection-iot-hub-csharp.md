---
title: 使用 C# 为 Azure 数据资源管理器创建 IoT 中心数据连接
description: 本文介绍如何使用 C# 为 Azure 数据资源管理器创建 IoT 中心数据连接。
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 10/07/2019
ms.date: 09/30/2020
ms.openlocfilehash: e57bddfc3961e2048487a2dc654aa3bcfca949b4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105162"
---
# <a name="create-an-iot-hub-data-connection-for-azure-data-explorer-by-using-c-preview"></a>使用 C#（预览版）为 Azure 数据资源管理器创建 IoT 中心数据连接

> [!div class="op_single_selector"]
> * [门户](ingest-data-iot-hub.md)
> * [C#](data-connection-iot-hub-csharp.md)
> * [Python](data-connection-iot-hub-python.md)
> * [Azure Resource Manager 模板](data-connection-iot-hub-resource-manager.md)

[!INCLUDE [data-connector-intro](includes/data-connector-intro.md)]
在本文中，你将使用 C# 为 Azure 数据资源管理器创建 IoT 中心数据连接。

## <a name="prerequisites"></a>先决条件

* 如果尚未安装 Visual Studio 2019，可以下载并使用 **免费的** [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。
* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* 创建[群集和数据库](create-cluster-database-csharp.md)
* 创建[表和列映射](./net-sdk-ingest-data.md#create-a-table-on-your-test-cluster)
* 设置[数据库和表策略](database-table-policies-csharp.md)（可选）
* 创建[配置了共享访问策略的 IoT 中心](ingest-data-iot-hub.md#create-an-iot-hub)。

[!INCLUDE [data-explorer-data-connection-install-nuget-csharp](includes/data-explorer-data-connection-install-nuget-csharp.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-iot-hub-data-connection"></a>添加 IoT 中心数据连接 

以下示例演示如何以编程方式添加 IoT 中心数据连接。 请参阅[将 Azure 数据资源管理器表连接到 IoT 中心](ingest-data-iot-hub.md#connect-azure-data-explorer-table-to-iot-hub)，了解如何使用 Azure 门户添加 IoT 中心数据连接。

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
//The IoT hub that is created as part of the Prerequisites
var iotHubResourceId = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.Devices/IotHubs/xxxxxx";
var sharedAccessPolicyName = "iothubforread";
var consumerGroup = "$Default";
var location = "China East 2";
//The table and column mapping are created as part of the Prerequisites
var tableName = "StormEvents";
var mappingRuleName = "StormEvents_CSV_Mapping";
var dataFormat = DataFormat.CSV;

await kustoManagementClient.DataConnections.CreateOrUpdate(resourceGroupName, clusterName, databaseName, dataConnectionName,
            new IotHubDataConnection(iotHubResourceId, consumerGroup, sharedAccessPolicyName, tableName: tableName, location: location, mappingRuleName: mappingRuleName, dataFormat: dataFormat));
```

|**设置** | **建议的值** | **字段说明**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 租户 ID。 也称为目录 ID。|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 用于创建资源的订阅 ID。|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 可以访问租户中资源的应用程序的客户端 ID。|
| clientSecret | *xxxxxxxxxxxxxx* | 可以访问租户中资源的应用程序的客户端密码。 |
| resourceGroupName | *testrg* | 包含群集的资源组的名称。|
| clusterName | mykustocluster  | 群集的名称。|
| databaseName | mykustodatabase  | 群集中目标数据库的名称。|
| dataConnectionName | *myeventhubconnect* | 所需的数据连接名称。|
| tableName | *StormEvents* | 目标数据库中目标表的名称。|
| mappingRuleName | *StormEvents_CSV_Mapping* | 与目标表相关的列映射的名称。|
| dataFormat | *csv* | 消息的数据格式。|
| iotHubResourceId | 资源 ID | 包含要引入的数据的 IoT 中心的资源 ID。 |
| sharedAccessPolicyName | *iothubforread* | 共享访问策略的名称，这些策略定义设备与服务连接到 IoT 中心所需的权限。 |
| consumerGroup | *$Default* | 事件中心的使用者组。|
| location | *中国东部 2* | 数据连接资源的位置。|

[!INCLUDE [data-explorer-data-connection-clean-resources-csharp](includes/data-explorer-data-connection-clean-resources-csharp.md)]
