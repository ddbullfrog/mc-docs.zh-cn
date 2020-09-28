---
title: 使用 C# 在 Azure 数据资源管理器群集上配置流式引入
description: 了解如何配置 Azure 数据资源管理器群集，并开始使用 C# 通过流式引入加载数据
author: orspod
ms.author: v-tawe
ms.reviewer: alexefro
ms.service: data-explorer
ms.topic: how-to
origin.date: 07/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: 9d403bc150a2c25f36473cfc9ffd493b43d5225b
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146579"
---
# <a name="configure-streaming-ingestion-on-your-azure-data-explorer-cluster-using-c"></a>使用 C# 在 Azure 数据资源管理器群集上配置流式引入

> [!div class="op_single_selector"]
> * [Portal](ingest-data-streaming.md)
> * [C#](ingest-data-streaming-csharp.md)

[!INCLUDE [ingest-data-streaming-intro](includes/ingest-data-streaming-intro.md)]

## <a name="prerequisites"></a>先决条件

* 如果尚未安装 Visual Studio 2019，可以下载并使用“免费的”[Visual Studio 2019 社区版](https://www.visualstudio.com/downloads/)。  在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。
* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 创建 [Azure 数据资源管理器群集和数据库](create-cluster-database-csharp.md)
   
## <a name="enable-streaming-ingestion-on-your-cluster-using-c"></a>使用 C# 在群集上启用流式引入

### <a name="enable-streaming-ingestion-while-creating-a-new-cluster"></a>创建新群集时启用流式引入

可以在创建新的 Azure 数据资源管理器群集时启用流式引入。

```csharp
...
var cluster = new Cluster(location, sku, enableStreamingIngest:true);
...
```

### <a name="enable-streaming-ingestion-on-an-existing-cluster"></a>在现有群集上启用流式引入

若要在 Azure Data Explorer 群集上启用流式引入，请运行以下代码：

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
    var clusterName = "mystreamingcluster";
    var clusterUpdateParameters = new ClusterUpdate(enableStreamingIngest: true);

    await kustoManagementClient.Clusters.UpdateAsync(resourceGroupName, clusterName, clusterUpdateParameters);
```

> [!WARNING]
> 启用流式引入之前，请查看[限制](#limitations)。

## <a name="create-a-target-table-and-define-the-policy-using-c"></a>使用 C# 创建目标表并定义策略

若要创建表并在此表上定义流式引入策略，请运行以下代码：

```csharp
    var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
    var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
    var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
    var databaseName = "StreamingTestDb";
    var tableName = "TestTable";
    var kcsb = new KustoConnectionStringBuilder("https://mystreamingcluster.chinaeast2.kusto.chinacloudapi.cn", databaseName);
    kcsb = kcsb.WithAadApplicationKeyAuthentication(clientId, clientSecret, tenantId);

    var tableSchema = new TableSchema(
        tableName,
        new ColumnSchema[]
        {
            new ColumnSchema("TimeStamp", "System.DateTime"),
            new ColumnSchema("Name",      "System.String"),
            new ColumnSchema("Metric",    "System.int"),
            new ColumnSchema("Source",    "System.String"),
        });

    var tableCreateCommand = CslCommandGenerator.GenerateTableCreateCommand(tableSchema);
    var tablePolicyAlterCommand = CslCommandGenerator.GenerateTableAlterStreamingIngestionPolicyCommand(tableName, isEnabled: true);
    using (var client = KustoClientFactory.CreateCslAdminProvider(kcsb))
    {
        client.ExecuteControlCommand(tableCreateCommand);

        client.ExecuteControlCommand(tablePolicyAlterCommand);
    }
```

[!INCLUDE [ingest-data-streaming-use](includes/ingest-data-streaming-types.md)]

[!INCLUDE [ingest-data-streaming-disable](includes/ingest-data-streaming-disable.md)]

### <a name="drop-the-streaming-ingestion-policy-using-c"></a>使用 C# 删除流式引入策略

若要从此表删除流式引入策略，请运行以下代码：
    
```csharp
        var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
        var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
        var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
        var databaseName = "StreamingTestDb";
        var tableName = "TestTable";
        var kcsb = new KustoConnectionStringBuilder("https://mystreamingcluster.chinaeast2.kusto.chinacloudapi.cn", databaseName);
        kcsb = kcsb.WithAadApplicationKeyAuthentication(clientId, clientSecret, tenantId);
    
        var tablePolicyDropCommand = CslCommandGenerator.GenerateTableStreamingIngestionPolicyDropCommand(databaseName, tableName);
        using (var client = KustoClientFactory.CreateCslAdminProvider(kcsb))
        {
            client.ExecuteControlCommand(tablePolicyDropCommand);
        }
```

### <a name="disable-streaming-ingestion-using-c"></a>使用 C# 禁用流式引入

若要在群集上禁用流式引入策略，请运行以下代码：
    
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
        var clusterName = "mystreamingcluster";
        var clusterUpdateParameters = new ClusterUpdate(enableStreamingIngest: false);
    
        await kustoManagementClient.Clusters.UpdateAsync(resourceGroupName, clusterName, clusterUpdateParameters);
```
    
[!INCLUDE [ingest-data-streaming-limitations](includes/ingest-data-streaming-limitations.md)]

## <a name="next-steps"></a>后续步骤

* [在 Azure 数据资源管理器中查询数据](web-query-data.md)
