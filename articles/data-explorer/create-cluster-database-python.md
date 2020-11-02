---
title: 使用 Python 创建 Azure 数据资源管理器群集和数据库
description: 了解如何使用 Python 创建 Azure 数据资源管理器群集和数据库。
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 06/03/2019
ms.date: 09/30/2020
ms.openlocfilehash: 2df42752c9b5752b9f3f4f2637fcf7c303598b2c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105168"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-python"></a>使用 Python 创建 Azure 数据资源管理器群集和数据库

> [!div class="op_single_selector"]
> * [门户](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Go](create-cluster-database-go.md)
> * [ARM 模板](create-cluster-database-resource-manager.md)

本文将使用 Python 创建 Azure 数据资源管理器群集和数据库。 Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站和 IoT 设备等资源流式传输的海量数据。 若要使用 Azure 数据资源管理器，请先创建一个群集，并在该群集中创建一个或多个数据库。 然后将数据引入或加载到数据库，以便针对其运行查询。

## <a name="prerequisites"></a>必备条件

如果没有 Azure 订阅，可在开始前创建一个 [试用帐户](https://www.azure.cn/pricing/1rmb-trial) 。

* [Python 3.4+](https://www.python.org/downloads/)。

* [可以访问资源的 Azure AD 应用程序和服务主体](https://docs.azure.cn/active-directory/develop/howto-create-service-principal-portal)。 获取 `Directory (tenant) ID`、`Application ID` 和 `Client Secret` 的值。

## <a name="install-python-package"></a>安装 Python 包

要为 Azure 数据资源管理器 (Kusto) 安装 Python 包，请打开其路径中包含 Python 的命令提示符。 运行以下命令：

```
pip install azure-common
pip install azure-mgmt-kusto
```
## <a name="authentication"></a>Authentication
为了运行本文中的示例，我们需要可以访问资源的 Azure AD 应用程序和服务主体。 查看[创建 Azure AD 应用程序](https://docs.azure.cn/active-directory/develop/howto-create-service-principal-portal)以创建免费的 Azure AD 应用程序，并在订阅范围内添加角色分配。 它还演示如何获取 `Directory (tenant) ID`、`Application ID` 和 `Client Secret`。

## <a name="create-the-azure-data-explorer-cluster"></a>创建 Azure 数据资源管理器群集

1. 请使用以下命令创建群集：

    ```Python
    from azure.mgmt.kusto import KustoManagementClient
    from azure.mgmt.kusto.models import Cluster, AzureSku
    from azure.common.credentials import ServicePrincipalCredentials

    #Directory (tenant) ID
    tenant_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    #Application ID
    client_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    #Client Secret
    client_secret = "xxxxxxxxxxxxxx"
    subscription_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    credentials = ServicePrincipalCredentials(
        client_id=client_id,
        secret=client_secret,
        tenant=tenant_id
    )

    location = 'China East 2'
    sku_name = 'Standard_D13_v2'
    capacity = 5
    tier = "Standard"
    resource_group_name = 'testrg'
    cluster_name = 'mykustocluster'
    cluster = Cluster(location=location, sku=AzureSku(name=sku_name, capacity=capacity, tier=tier))
    
    kusto_management_client = KustoManagementClient(credentials, subscription_id)

    cluster_operations = kusto_management_client.clusters
    
    poller = cluster_operations.create_or_update(resource_group_name, cluster_name, cluster)
    poller.wait()
    ```

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | cluster_name | mykustocluster  | 所需的群集名称。|
   | sku_name | *Standard_D13_v2* | 将用于群集的 SKU。 |
   | 层 | *Standard* | SKU 层。 |
   | 容量 | *数字* | 群集实例的数目。 |
   | resource_group_name | *testrg* | 将在其中创建群集的资源组名称。 |

    > [!NOTE]
    > **创建群集** 是一个长时间运行的操作。 **create_or_update** 方法返回 LROPoller 的实例，请参阅 [LROPoller 类](https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python)获取详细信息。

1. 运行以下命令，检查群集是否已成功创建：

    ```Python
    cluster_operations.get(resource_group_name = resource_group_name, cluster_name= cluster_name, custom_headers=None, raw=False)
    ```

如果结果包含带 `Succeeded` 值的 `provisioningState`，则表示已成功创建群集。

## <a name="create-the-database-in-the-azure-data-explorer-cluster"></a>在 Azure 数据资源管理器群集中创建数据库

1. 请使用以下命令创建数据库：

    ```Python
    from azure.mgmt.kusto import KustoManagementClient
    from azure.common.credentials import ServicePrincipalCredentials
    from azure.mgmt.kusto.models import ReadWriteDatabase
    from datetime import timedelta

    #Directory (tenant) ID
    tenant_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    #Application ID
    client_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    #Client Secret
    client_secret = "xxxxxxxxxxxxxx"
    subscription_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
    credentials = ServicePrincipalCredentials(
        client_id=client_id,
        secret=client_secret,
        tenant=tenant_id
    )
    
    location = 'Central US'
    resource_group_name = 'testrg'
    cluster_name = 'mykustocluster'
    soft_delete_period = timedelta(days=3650)
    hot_cache_period = timedelta(days=3650)
    database_name = "mykustodatabase"

    kusto_management_client = KustoManagementClient(credentials, subscription_id)
    
    database_operations = kusto_management_client.databases
    database = ReadWriteDatabase(location=location,
                        soft_delete_period=soft_delete_period,
                        hot_cache_period=hot_cache_period)
    
    poller = database_operations.create_or_update(resource_group_name = resource_group_name, cluster_name = cluster_name, database_name = database_name, parameters = database)
    poller.wait()
    ```

    > [!NOTE]
    > 如果使用的是 Python 版本 0.4.0 或更低版本，请使用 Database 而不是 ReadWriteDatabase。

   |**设置** | **建议的值** | **字段说明**|
   |---|---|---|
   | cluster_name | mykustocluster  | 将在其中创建数据库的群集的名称。|
   | database_name | mykustodatabase  | 数据库名称。|
   | resource_group_name | *testrg* | 将在其中创建群集的资源组名称。 |
   | soft_delete_period | *3650 天，0:00:00* | 供查询使用的数据的保留时间。 |
   | hot_cache_period | *3650 天，0:00:00* | 数据将在缓存中保留的时间。 |

1. 若要查看已创建的数据库，请运行以下命令：

    ```Python
    database_operations.get(resource_group_name = resource_group_name, cluster_name = cluster_name, database_name = database_name)
    ```

现在，你有了一个群集和一个数据库。

## <a name="clean-up-resources"></a>清理资源

* 如果计划学习我们的其他文章，请保留已创建的资源。
* 若要清理资源，请删除群集。 删除群集时，也会删除其中的所有数据库。 使用以下命令删除群集：

    ```Python
    cluster_operations.delete(resource_group_name = resource_group_name, cluster_name = cluster_name)
    ```

## <a name="next-steps"></a>后续步骤

* [使用 Azure 数据资源管理器 Python 库引入数据](python-ingest-data.md)
