---
title: 使用 Azure 数据资源管理器 C# SDK 创建策略
description: 本文介绍如何使用 C# 创建策略。
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 09/24/2019
ms.date: 09/30/2020
ms.openlocfilehash: 4813a347a3090f65020c3ea89c91d379cc56aa31
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103561"
---
# <a name="create-database-and-table-policies-for-azure-data-explorer-by-using-c"></a>使用 C# 为 Azure 数据资源管理器创建数据库和表策略

> [!div class="op_single_selector"]
> * [C#](database-table-policies-csharp.md)
> * [Python](database-table-policies-python.md)
>

Azure 数据资源管理器是一项快速且高度可缩放的数据探索服务，适用于日志和遥测数据。 在本文中，你将使用 C# 为 Azure 数据资源管理器创建数据库和表策略。

## <a name="prerequisites"></a>必备条件

* Visual Studio 2019。 如果没有 Visual Studio 2019，可以下载并使用 *免费的* [Visual Studio Community 2019](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保选择“Azure 开发”。 
* Azure 订阅。 可以根据需要在开始之前创建[试用版 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* [一个测试群集和数据库](create-cluster-database-csharp.md)。
* [测试表](./net-sdk-ingest-data.md#create-a-table-on-your-test-cluster)。

## <a name="install-c-nuget"></a>安装 C# NuGet

* 安装 [Azure 数据资源管理器 (Kusto) NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)。
* 安装 [Microsoft.Azure.Kusto.Data.NETStandard NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Data.NETStandard/)。 （可选，用于更改表策略。）
* 安装 [Microsoft.IdentityModel.Clients.ActiveDirectory NuGet 包](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)以进行身份验证。

## <a name="authentication"></a>Authentication
若要运行本文中的示例，需要可以访问资源的 Azure Active Directory (Azure AD) 应用程序和服务主体。 可以使用相同的 Azure AD 应用程序通过[测试群集和数据库](create-cluster-database-csharp.md#authentication)进行身份验证。 如果要使用其他 Azure AD 应用程序，请参阅[创建 Azure AD 应用程序](/active-directory/develop/howto-create-service-principal-portal)以创建免费的 Azure AD 应用程序并在订阅范围内添加角色分配。 本文还演示如何获取 `Directory (tenant) ID`、`Application ID` 和 `Client secret`。 可能需要将新的 Azure AD 应用程序添加为数据库中的主体。 有关详细信息，请参阅[管理 Azure 数据资源管理器数据库权限](manage-database-permissions.md)。

## <a name="alter-database-retention-policy"></a>更改数据库保留策略
使用 10 天的软删除期设置保留策略。
    
```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
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
//The cluster and database that are created as part of the prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
await kustoManagementClient.Databases.UpdateAsync(resourceGroupName, clusterName, databaseName, new DatabaseUpdate(softDeletePeriod: TimeSpan.FromDays(10)));
```

## <a name="alter-database-cache-policy"></a>更改数据库缓存策略
为数据库设置缓存策略。 头五天，数据将位于群集 SSD 上。

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
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
//The cluster and database that are created as part of the prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
await kustoManagementClient.Databases.UpdateAsync(resourceGroupName, clusterName, databaseName, new DatabaseUpdate(hotCachePeriod: TimeSpan.FromDays(5)));
```

## <a name="alter-table-cache-policy"></a>更改表缓存策略
为表设置缓存策略。 头五天，数据将位于群集 SSD 上。

```csharp
var kustoUri = "https://<ClusterName>.<Region>.kusto.chinacloudapi.cn:443/";
var databaseName = "<DatabaseName>";
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
var tableName = "<TableName>"

var kustoConnectionStringBuilder =
    new KustoConnectionStringBuilder(kustoUri)
    {
        FederatedSecurity = true,
        InitialCatalog = databaseName,
        ApplicationClientId = clientId,
        ApplicationKey = clientSecret,
        Authority = tenantId
    };

using (var kustoClient = KustoClientFactory.CreateCslAdminProvider(kustoConnectionStringBuilder))
{
    //dataHotSpan and indexHotSpan should have the same value
    var hotSpan = TimeSpan.FromDays(5);
    var command1 = CslCommandGenerator.GenerateAlterTableCachingPolicyCommand(tableName: tableName,
                    dataHotSpan: hotSpan, indexHotSpan: hotSpan);

    kustoClient.ExecuteControlCommand(command);
}
```

## <a name="add-a-new-principal-for-the-database"></a>为数据库添加新主体
添加新的 Azure AD 应用程序作为数据库的管理员主体。

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var clientIdToAdd = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
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
//The cluster and database that are created as part of the prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
await kustoManagementClient.Databases.AddPrincipalsAsync(resourceGroupName, clusterName, databaseName,
                new DatabasePrincipalListRequest()
                {
                    Value = new List<DatabasePrincipal>()
                    {
                        new DatabasePrincipal("Admin", "<database_principle_name>", "App", appId: clientIdToAdd, tenantName:tenantId)
                    }
                });
```
## <a name="next-steps"></a>后续步骤

* [阅读有关数据库和表策略的更多信息](kusto/management/policies.md)
