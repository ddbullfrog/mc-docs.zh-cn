---
title: 使用 Python 为 Azure 数据资源管理器创建事件中心数据连接
description: 本文介绍如何使用 Python 为 Azure 数据资源管理器创建事件中心数据连接。
author: orspod
ms.author: v-tawe
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
origin.date: 10/07/2019
ms.date: 09/24/2020
ms.openlocfilehash: 52e87910440e95bb09d39ebf488ae8e13cfc5d00
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146573"
---
# <a name="create-an-event-hub-data-connection-for-azure-data-explorer-by-using-python"></a>使用 Python 为 Azure 数据资源管理器创建事件中心数据连接

> [!div class="op_single_selector"]
> * [门户](ingest-data-event-hub.md)
> * [C#](data-connection-event-hub-csharp.md)
> * [Python](data-connection-event-hub-python.md)
> * [Azure Resource Manager 模板](data-connection-event-hub-resource-manager.md)

[!INCLUDE [data-connector-intro](includes/data-connector-intro.md)]
在本文中，你将使用 Python 为 Azure 数据资源管理器创建事件中心数据连接。 

## <a name="prerequisites"></a>先决条件

* 具有活动订阅的 Azure 帐户。 [创建试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* [Python 3.4+](https://www.python.org/downloads/)。
* [一个群集和数据库](create-cluster-database-python.md)。
* [表和列映射](net-standard-ingest-data.md#create-a-table-on-your-test-cluster)。
* [数据库和表策略](database-table-policies-python.md)（可选）。
* [包含要引入数据的事件中心](ingest-data-event-hub.md#create-an-event-hub)。

[!INCLUDE [data-explorer-data-connection-install-package-python](includes/data-explorer-data-connection-install-package-python.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-event-hub-data-connection"></a>添加事件中心数据连接

以下示例演示如何以编程方式添加事件中心数据连接。 请参阅[连接到事件中心](ingest-data-event-hub.md#connect-to-the-event-hub)，了解如何使用 Azure 门户添加事件中心数据连接。

```Python
from azure.mgmt.kusto import KustoManagementClient
from azure.mgmt.kusto.models import EventHubDataConnection
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
kusto_management_client = KustoManagementClient(credentials, subscription_id)

resource_group_name = "testrg"
#The cluster and database that are created as part of the Prerequisites
cluster_name = "mykustocluster"
database_name = "mykustodatabase"
data_connection_name = "myeventhubconnect"
#The event hub that is created as part of the Prerequisites
event_hub_resource_id = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.EventHub/namespaces/xxxxxx/eventhubs/xxxxxx";
consumer_group = "$Default"
location = "China East 2"
#The table and column mapping that are created as part of the Prerequisites
table_name = "StormEvents"
mapping_rule_name = "StormEvents_CSV_Mapping"
data_format = "csv"
#Returns an instance of LROPoller, check https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python
poller = kusto_management_client.data_connections.create_or_update(resource_group_name=resource_group_name, cluster_name=cluster_name, database_name=database_name, data_connection_name=data_connection_name,
                                        parameters=EventHubDataConnection(event_hub_resource_id=event_hub_resource_id, consumer_group=consumer_group, location=location,
                                                                            table_name=table_name, mapping_rule_name=mapping_rule_name, data_format=data_format))
```

|**设置** | **建议的值** | **字段说明**|
|---|---|---|
| tenant_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 租户 ID。 也称为目录 ID。|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 用于创建资源的订阅 ID。|
| client_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | 可以访问租户中资源的应用程序的客户端 ID。|
| client_secret | *xxxxxxxxxxxxxx* | 可以访问租户中资源的应用程序的客户端密码。 |
| resource_group_name | *testrg* | 包含群集的资源组的名称。|
| cluster_name | mykustocluster** | 群集的名称。|
| database_name | mykustodatabase** | 群集中目标数据库的名称。|
| data_connection_name | *myeventhubconnect* | 所需的数据连接名称。|
| table_name | *StormEvents* | 目标数据库中目标表的名称。|
| mapping_rule_name | *StormEvents_CSV_Mapping* | 与目标表相关的列映射的名称。|
| data_format | *csv* | 消息的数据格式。|
| event_hub_resource_id | 资源 ID | 包含要引入的数据的事件中心的资源 ID。 |
| consumer_group | *$Default* | 事件中心的使用者组。|
| location | *中国东部 2* | 数据连接资源的位置。|

[!INCLUDE [data-explorer-data-connection-clean-resources-python](includes/data-explorer-data-connection-clean-resources-python.md)]
