---
title: 使用 C# 向 Azure 数据资源管理器添加群集主体
description: 本文介绍如何使用 C# 向 Azure 数据资源管理器添加群集主体。
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 02/03/2020
ms.date: 09/24/2020
ms.openlocfilehash: 84812939451ef327948ffb6ae1aeb69cb77f5513
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146770"
---
# <a name="add-cluster-principals-for-azure-data-explorer-by-using-c"></a>使用 C# 向 Azure 数据资源管理器添加群集主体

> [!div class="op_single_selector"]
> * [C#](cluster-principal-csharp.md)
> * [Python](cluster-principal-python.md)
> * [Azure Resource Manager 模板](cluster-principal-resource-manager.md)

Azure 数据资源管理器是一项快速且高度可缩放的数据探索服务，适用于日志和遥测数据。 在本文中，将使用 C# 向 Azure 数据资源管理器添加群集主体。

## <a name="prerequisites"></a>必备条件

* 如果尚未安装 Visual Studio 2019，可以下载并使用**免费的** [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。 
* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* [创建群集](create-cluster-database-csharp.md)。

## <a name="install-c-nuget"></a>安装 C# NuGet

* 安装 [Microsoft.Azure.Management.kusto](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)。
* 安装 [Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication) 以进行身份验证。

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-a-cluster-principal"></a>添加群集主体

以下示例演示如何以编程方式添加群集主体。

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";

var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, clientSecret);
var kustoManagementClient = new KustoManagementClient(serviceCreds)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster that is created as part of the Prerequisites
var clusterName = "mykustocluster";
string principalAssignmentName = "clusterPrincipalAssignment1";
string principalId = "xxxxxxxx";//User email, application ID, or security group name
string role = "AllDatabasesAdmin";//AllDatabasesAdmin or AllDatabasesViewer
string tenantIdForPrincipal = tenantId;
string principalType = "App";//User, App, or Group

var clusterPrincipalAssignment = new ClusterPrincipalAssignment(principalId, role, principalType, tenantId: tenantIdForPrincipal);
await kustoManagementClient.ClusterPrincipalAssignments.CreateOrUpdateAsync(resourceGroupName, clusterName, principalAssignmentName, clusterPrincipalAssignment);
```

|**设置** | **建议的值** | **字段说明**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 租户 ID。 也称为目录 ID。|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 用于创建资源的订阅 ID。|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 可以访问租户中资源的应用程序的客户端 ID。|
| clientSecret | *xxxxxxxxxxxxxx* | 可以访问租户中资源的应用程序的客户端密码。 |
| resourceGroupName | *testrg* | 包含群集的资源组的名称。|
| clusterName | mykustocluster  | 群集的名称。|
| principalAssignmentName | *clusterPrincipalAssignment1* | 群集主体资源的名称。|
| principalId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 主体 ID，可以是用户电子邮件、应用程序 ID 或安全组名称。|
| 角色 (role) | *AllDatabasesAdmin* | 群集主体的角色，可以是“AllDatabasesAdmin”或“AllDatabasesViewer”。|
| tenantIdForPrincipal | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 主体的租户 ID。|
| principalType | *应用* | 主体的类型，可以是“User”、“App”或“Group”|

## <a name="next-steps"></a>后续步骤

* [添加数据库主体](database-principal-csharp.md)
