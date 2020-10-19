---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 实例池 API - Azure Databricks
description: 了解 Databricks 实例池 API。
ms.openlocfilehash: 0c0f71ba2707a6c26fdee0703f7b18e837e517d3
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937677"
---
# <a name="instance-pools-api"></a>实例池 API

利用实例池 API，可以创建、编辑、删除和列出实例池。

实例池通过维护一组空闲的随时可用的云实例来减少群集启动和自动缩放的时间。 如果附加到池的群集需要一个实例，它首先会尝试分配池中的一个空闲实例。 如果池中没有空闲实例，则该池会通过从实例提供程序分配新的实例进行扩展，以满足群集的请求。 当群集释放了某个实例时，该实例将返回到池中，以供其他群集使用。 只有附加到池的群集才能使用该池的空闲实例。

当实例在池中处于空闲状态时，Azure Databricks 不会收取 DBU 费用， 但这会产生实例提供程序费用，具体请参阅[定价](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="create"></a><a id="clusterinstancepoolservicecreateinstancepool"> </a><a id="create"> </a>创建

| 端点                          | HTTP 方法     |
|-----------------------------------|-----------------|
| `2.0/instance-pools/create`       | `POST`          |

创建实例池。 使用返回的 `instance_pool_id` 来查询实例池的状态，其中包括池当前分配的实例数。 如果提供了 `min_idle_instances` 参数，实例会在后台进行预配，在 [InstancePoolStats](#clusterinstancepoolstats) 中的 `idle_count` 等于已请求的最小值后即可使用。

> [!NOTE]
>
> 由于实例提供程序限制或暂时性的网络问题，Azure Databricks 可能无法获取一些已请求的空闲实例。 群集仍可附加到实例池，但可能无法像原来那样快速启动。

示例请求：

```json
{
  "instance_pool_name": "my-pool",
  "node_type_id": "Standard_D3_v2",
  "min_idle_instances": 10
}
```

响应：

```json
{
  "instance_pool_id": "0101-120000-brick1-pool-ABCD1234"
}
```

### <a name="request-structure"></a><a id="clustercreateinstancepool"> </a><a id="request-structure"> </a>请求结构

| 字段名称                                | 类型                                                                   | 描述                                                                                                                                                                                                                                                                                                                                                                 |
|-------------------------------------------|------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| instance_pool_name                        | `STRING`                                                               | 实例池的名称。 这是执行创建和编辑操作所必需的。 它必须独一无二、非空且少于 100 个字符。                                                                                                                                                                                                                                 |
| min_idle_instances                        | `INT32`                                                                | 由池维护的空闲实例的最小数目。 这是对活动群集正在使用的任何实例的补充。                                                                                                                                                                                                                                                |
| max_capacity                              | `INT32`                                                                | 池可以包含的实例的最大数目，包括空闲实例数和群集正在使用的实例数。 达到最大容量后，你将无法通过该池创建新的群集，并且现有的群集将无法自动纵向扩展，直到通过群集终止或纵向缩减操作让池中的某些实例进入空闲状态。                                       |
| node_type_id                              | `STRING`                                                               | 池中实例的节点类型。 附加到池的所有群集都继承此节点类型，池的空闲实例根据此类型进行分配。 可以使用[列出节点类型](clusters.md#clusterclusterservicelistnodetypes) API 调用来检索可用节点类型的列表。                                                              |
| custom_tags                               | [ClusterTag](clusters.md#clusterclustertag) 的数组                | 实例池资源的其他标记。 Azure Databricks 会使用这些标记以及 default_tags 来标记所有的池资源（例如，VM 磁盘卷）。<br><br>Azure Databricks 最多允许 41 个自定义标记。                                                                                                                                                              |
| idle_instance_autotermination_minutes     | `INT32`                                                                | 超出 min_idle_instances 的空闲实例在被终止之前由池维护的分钟数。 如果未指定此项，则会在默认的超时期限过后自动终止额外的空闲实例。 如果指定此项，则时间必须在 0 到 10000 分钟之间。 如果提供 0，系统会尽快删除多余的空闲实例。 |
| enable_elastic_disk                       | `BOOL`                                                                 | 自动缩放本地存储：启用后，池中的实例在磁盘空间不足时会动态获取更多磁盘空间。                                                                                                                                                                                                                       |
| disk_spec                                 | [DiskSpec](#clusterdiskspec)                                           | 定义附加到池中每个实例的初始远程存储的容量。                                                                                                                                                                                                                                                                                         |
| preloaded_spark_versions                  | 一个由 `STRING` 构成的数组                                                   | 一个列表，其中包含池最多在每个实例上安装一个的运行时版本。 使用预先加载的运行时版本的池群集启动速度更快，因为不需等待映像下载。 可以通过使用[运行时版本](clusters.md#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。                          |

### <a name="response-structure"></a><a id="clustercreateinstancepoolresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称           | 类型           | 描述                              |
|----------------------|----------------|------------------------------------------|
| instance_pool_id     | `STRING`       | 已创建的实例池的 ID。     |

## <a name="edit"></a><a id="clusterinstancepoolserviceeditinstancepool"> </a><a id="edit"> </a>编辑

| 端点                        | HTTP 方法     |
|---------------------------------|-----------------|
| `2.0/instance-pools/edit`       | `POST`          |

编辑实例池。 这会修改现有实例池的配置。

> [!NOTE]
>
> * 仅可编辑以下字段：`instance_pool_name`、`min_idle_instances`、`max_capacity` 和 `idle_instance_autotermination_minutes`。
> * 必须提供 `instance_pool_name`。
> * 必须提供 `node_type_id`，并且它必须与原始 `node_type_id` 匹配。

示例请求：

```json
{
  "instance_pool_id": "0101-120000-brick1-pool-ABCD1234",
  "instance_pool_name": "my-edited-pool",
  "node_type_id": "Standard_D3_v2",
  "min_idle_instances": 5,
  "max_capacity": 200,
  "idle_instance_autotermination_minutes": 30
}
```

### <a name="request-structure"></a><a id="clustereditinstancepool"> </a><a id="request-structure"> </a>请求结构

| 字段名称                                | 类型                                        | 描述                                                                                                                                                                                                                                                                                                                                                                   |
|-------------------------------------------|---------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| instance_pool_id                          | `STRING`                                    | 要编辑的实例池的 ID。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                  |
| instance_pool_name                        | `STRING`                                    | 实例池的名称。 这是执行创建和编辑操作所必需的。 它必须独一无二、非空且少于 100 个字符。                                                                                                                                                                                                                                   |
| min_idle_instances                        | `INT32`                                     | 由池维护的空闲实例的最小数目。 这是对活动群集正在使用的任何实例的补充。                                                                                                                                                                                                                                                  |
| max_capacity                              | `INT32`                                     | 池可以包含的实例的最大数目，包括空闲实例数和群集正在使用的实例数。 达到最大容量后，你将无法通过该池创建新的群集，并且现有的群集将无法自动纵向扩展，直到通过群集终止或纵向缩减操作让池中的某些实例进入空闲状态。                                         |
| node_type_id                              | `STRING`                                    | 池中实例的节点类型。 附加到池的所有群集都继承此节点类型，池的空闲实例根据此类型进行分配。 可以使用[列出节点类型](clusters.md#clusterclusterservicelistnodetypes) API 调用来检索可用节点类型的列表。                                                                |
| idle_instance_autotermination_minutes     | `INT32`                                     | 超出 `min_idle_instances` 的空闲实例在被终止之前由池维护的分钟数。 如果未指定此项，则会在默认的超时期限过后自动终止额外的空闲实例。 如果指定此项，则时间必须在 0 到 10000 分钟之间。 如果提供 0，系统会尽快删除多余的空闲实例。 |

## <a name="delete"></a><a id="clusterinstancepoolservicedeleteinstancepool"> </a><a id="delete"> </a>删除

| 端点                          | HTTP 方法     |
|-----------------------------------|-----------------|
| `2.0/instance-pools/delete`       | `POST`          |

删除实例池。 这会永久删除实例池。 池中的空闲实例会被异步终止。 新群集无法附加到池。 附加到池的处于运行状态的群集会继续运行，但无法自动纵向扩展。 附加到池的已终止群集将无法启动，除非将其编辑为不再使用池。

示例请求：

```json
{
  "instance_pool_id": "0101-120000-brick1-pool-ABCD1234"
}
```

### <a name="request-structure"></a><a id="clusterdeleteinstancepool"> </a><a id="request-structure"> </a>请求结构

| 字段名称           | 类型           | 描述                                |
|----------------------|----------------|--------------------------------------------|
| instance_pool_id     | `STRING`       | 要删除的实例池的 ID。     |

## <a name="get"></a><a id="clusterinstancepoolservicegetinstancepool"> </a><a id="get"> </a>获取

| 端点                       | HTTP 方法     |
|--------------------------------|-----------------|
| `2.0/instance-pools/get`       | `GET`           |

根据标识符检索实例池的信息。

示例请求：

```bash
/instance-pools/get?instance_pool_id=0101-120000-brick1-pool-ABCD1234
```

示例响应：

```json
{
  "instance_pool_name": "mypool",
  "node_type_id": "Standard_D3_v2",
  "idle_instance_autotermination_minutes": 60,
  "enable_elastic_disk": false,
  "preloaded_spark_versions": [
    "5.4.x-scala2.11"
  ],
  "instance_pool_id": "101-120000-brick1-pool-ABCD1234",
  "default_tags": {
    "Vendor": "Databricks",
    "DatabricksInstancePoolCreatorId": "100125",
    "DatabricksInstancePoolId": "101-120000-brick1-pool-ABCD1234"
  },
  "state": "ACTIVE",
  "stats": {
    "used_count": 10,
    "idle_count": 5,
    "pending_used_count": 5,
    "pending_idle_count": 5
  },
  "status": {}
}
```

### <a name="request-structure"></a><a id="clustergetinstancepool"> </a><a id="request-structure"> </a>请求结构

| 字段名称           | 类型           | 描述                                                |
|----------------------|----------------|------------------------------------------------------------|
| instance_pool_id     | `STRING`       | 要检索其信息的实例池。     |

### <a name="response-structure"></a><a id="clustergetinstancepoolresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称                                | 类型                                                                   | 描述                                                                                                                                                                                                                                                                                                                                                                   |
|-------------------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| instance_pool_name                        | `STRING`                                                               | 实例池的名称。 这是执行创建和编辑操作所必需的。 它必须独一无二、非空且少于 100 个字符。                                                                                                                                                                                                                                   |
| min_idle_instances                        | `INT32`                                                                | 由池维护的空闲实例的最小数目。 这是对活动群集正在使用的任何实例的补充。                                                                                                                                                                                                                                                  |
| max_capacity                              | `INT32`                                                                | 池可以包含的实例的最大数目，包括空闲实例数和群集正在使用的实例数。 达到最大容量后，你将无法通过该池创建新的群集，并且现有的群集将无法自动纵向扩展，直到通过群集终止或纵向缩减操作让池中的某些实例进入空闲状态。                                         |
| node_type_id                              | `STRING`                                                               | 池中实例的节点类型。 附加到池的所有群集都继承此节点类型，池的空闲实例根据此类型进行分配。 可以使用[列出节点类型](clusters.md#clusterclusterservicelistnodetypes) API 调用来检索可用节点类型的列表。                                                                |
| custom_tags                               | [ClusterTag](clusters.md#clusterclustertag) 的数组                | 实例池资源的其他标记。 Azure Databricks 会使用这些标记以及 default_tags 来标记所有的池资源（例如，VM 磁盘卷）。<br><br>Azure Databricks 最多允许 41 个自定义标记。                                                                                                                                                                |
| idle_instance_autotermination_minutes     | `INT32`                                                                | 超出 `min_idle_instances` 的空闲实例在被终止之前由池维护的分钟数。 如果未指定此项，则会在默认的超时期限过后自动终止额外的空闲实例。 如果指定此项，则时间必须在 0 到 10000 分钟之间。 如果提供 0，系统会尽快删除多余的空闲实例。 |
| enable_elastic_disk                       | `BOOL`                                                                 | 自动缩放本地存储：启用后，池中的实例在磁盘空间不足时会动态获取更多磁盘空间。                                                                                                                                                                                                                         |
| disk_spec                                 | [DiskSpec](#clusterdiskspec)                                           | 定义附加到池中每个实例的初始远程存储的容量。                                                                                                                                                                                                                                                                                           |
| preloaded_spark_versions                  | 一个由 `STRING` 构成的数组                                                   | 一个列表，其中包含由池安装在每个实例上的运行时版本。 使用预先加载的运行时版本的池群集启动速度更快，因为不需等待映像下载。 可以通过使用[运行时版本](clusters.md#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。                                    |
| instance_pool_id                          | `STRING`                                                               | 实例池的符合规范的唯一标识符。                                                                                                                                                                                                                                                                                                                        |
| default_tags                              | [ClusterTag](clusters.md#clusterclustertag) 的数组                | 由 Azure Databricks 添加的标记（与任何 custom_tags 无关），其中包括：<br><br>* 供应商：Databricks<br>* DatabricksInstancePoolCreatorId：<create_user_id><br>* DatabricksInstancePoolId：<instance_pool_id>                                                                                                                                                        |
| state                                     | [InstancePoolState](#clusterinstancepoolstate)                         | 实例池的当前状态。                                                                                                                                                                                                                                                                                                                                           |
| stats                                     | [InstancePoolStats](#clusterinstancepoolstats)                         | 有关实例池使用情况的统计信息。                                                                                                                                                                                                                                                                                                                              |
| status                                    | [InstancePoolStatus](#clusterinstancepoolstatus)                       | 池中已失败的挂起实例的状态。                                                                                                                                                                                                                                                                                                                            |

## <a name="list"></a><a id="clusterinstancepoolservicelistinstancepools"> </a><a id="list"> </a>列出

| 端点                        | HTTP 方法     |
|---------------------------------|-----------------|
| `2.0/instance-pools/list`       | `GET`           |

列出所有实例池的信息。

示例响应：

```json
{
  "instance_pools": [
    {
      "instance_pool_name": "my-pool",
      "min_idle_instances": 10,
      "node_type_id": "Standard_D3_v2",
      "idle_instance_autotermination_minutes": 60,
      "instance_pool_id": "0101-120000-brick1-pool-ABCD1234",
      "default_tags": [
        { "DatabricksInstancePoolCreatorId", "1234" },
        { "DatabricksInstancePoolId", "0101-120000-brick1-pool-ABCD1234" }
      ],
      "stats": {
        "used_count": 10,
        "idle_count": 5,
        "pending_used_count": 5,
        "pending_idle_count": 5
      }
    }
  ]
}
```

### <a name="response-structure"></a><a id="clusterlistinstancepoolsresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称         | 类型                                                             | 描述                                                  |
|--------------------|------------------------------------------------------------------|--------------------------------------------------------------|
| instance_pools     | [InstancePoolAndStats](#clusterinstancepoolandstats) 的数组 | 实例池的列表，其中包含实例池的统计信息。     |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="instance-poolsadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [InstancePoolState](#instancepoolstate)
* [InstancePoolStats](#instancepoolstats)
* [InstancePoolStatus](#instancepoolstatus)
* [PendingInstanceError](#pendinginstanceerror)
* [DiskSpec](#diskspec)
* [DiskType](#disktype)
* [InstancePoolAndStats](#instancepoolandstats)
* [AzureDiskVolumeType](#azurediskvolumetype)

### <a name="instancepoolstate"></a><a id="clusterinstancepoolstate"> </a><a id="instancepoolstate"> </a>InstancePoolState

实例池的状态。 当前允许的状态转换为：

* `ACTIVE` -> `DELETED`

| 名称        | 描述                                                                                         |
|-------------|-----------------------------------------------------------------------------------------------------|
| ACTIVE      | 指示实例池处于活动状态。 群集可以附加到该实例池。                                    |
| DELETED     | 指示实例池已被删除，不再可供访问。                           |

### <a name="instancepoolstats"></a><a id="clusterinstancepoolstats"> </a><a id="instancepoolstats"> </a>InstancePoolStats

有关实例池使用情况的统计信息。

| 字段名称             | 类型          | 描述                                                         |
|------------------------|---------------|---------------------------------------------------------------------|
| used_count             | `INT32`       | 群集正在使用的活动实例数。            |
| idle_count             | `INT32`       | 群集未在使用的活动实例数。        |
| pending_used_count     | `INT32`       | 分配给群集的挂起实例数。         |
| pending_idle_count     | `INT32`       | 未分配给群集的挂起实例数。     |

### <a name="instancepoolstatus"></a><a id="clusterinstancepoolstatus"> </a><a id="instancepoolstatus"> </a>InstancePoolStatus

池中已失败的挂起实例的状态。

| 字段名称                | 类型                                                             | 描述                                                |
|---------------------------|------------------------------------------------------------------|------------------------------------------------------------|
| pending_instance_errors   | [PendingInstanceError](#clusterpendinginstanceerror) 的数组 | 失败的挂起实例的错误消息列表。   |

### <a name="pendinginstanceerror"></a><a id="clusterpendinginstanceerror"> </a><a id="pendinginstanceerror"> </a>PendingInstanceError

失败的挂起实例的错误消息。

| 字段名称             | 类型           | 描述                                                         |
|------------------------|----------------|---------------------------------------------------------------------|
| instance_id            | `STRING`       | 失败实例的 ID。                                          |
| message                | `STRING`       | 描述失败原因的消息。                        |

### <a name="diskspec"></a><a id="clusterdiskspec"> </a><a id="diskspec"> </a>DiskSpec

描述要附加到每个实例的初始磁盘集。 例如，如果有 3 个实例，一开始为每个实例配置 2 个磁盘，每个磁盘 100 GiB，则 Azure Databricks 为这些实例总共创建 6 个磁盘，每个磁盘 100 GiB。

| 字段名称     | 类型                         | 描述                                                                                                                                                                                                                                                                     |
|----------------|------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| disk_type      | [DiskType](#clusterdisktype) | 要附加的磁盘的类型。                                                                                                                                                                                                                                                    |
| disk_count     | `INT32`                      | 要附加到每个实例的磁盘数：<br><br>* 只会为支持的节点类型启用此功能。<br>* 用户可以选择的磁盘数量不能超过节点类型支持的磁盘数量上限。<br>* 对于不带本地磁盘的节点类型，需要指定至少一个磁盘。 |
| disk_size      | `INT32`                      | 要附加的每个磁盘的大小（以 GiB 为单位）。 值必须在特定实例类型支持的范围内：<br><br>* 高级 LRS (SSD)：1 - 1023 GiB<br>* 标准 LRS (HDD)：1- 1023 GiB                                                                               |

### <a name="disktype"></a><a id="clusterdisktype"> </a><a id="disktype"> </a>DiskType

描述磁盘的类型。

| 字段名称                                            | 类型                                                                 | 描述                         |
|-------------------------------------------------------|----------------------------------------------------------------------|-------------------------------------|
| azure_disk_volume_type                                | [AzureDiskVolumeType](#clusterazurediskvolumetypepools)              | 要使用的 Azure 磁盘的类型。      |

### <a name="instancepoolandstats"></a><a id="clusterinstancepoolandstats"> </a><a id="instancepoolandstats"> </a>InstancePoolAndStats

| 字段名称                                | 类型                                                                   | 描述                                                                                                                                                                                                                                                                                                                                                                   |
|-------------------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| instance_pool_name                        | `STRING`                                                               | 实例池的名称。 这是执行创建和编辑操作所必需的。 它必须独一无二、非空且少于 100 个字符。                                                                                                                                                                                                                                   |
| min_idle_instances                        | `INT32`                                                                | 由池维护的空闲实例的最小数目。 这是对活动群集正在使用的任何实例的补充。                                                                                                                                                                                                                                                  |
| max_capacity                              | `INT32`                                                                | 池可以包含的实例的最大数目，包括空闲实例数和群集正在使用的实例数。 达到最大容量后，你将无法通过该池创建新的群集，并且现有的群集将无法自动纵向扩展，直到通过群集终止或纵向缩减操作让池中的某些实例进入空闲状态。                                         |
| node_type_id                              | `STRING`                                                               | 池中实例的节点类型。 附加到池的所有群集都继承此节点类型，池的空闲实例根据此类型进行分配。 可以使用[列出节点类型](clusters.md#clusterclusterservicelistnodetypes) API 调用来检索可用节点类型的列表。                                                                |
| custom_tags                               | [ClusterTag](clusters.md#clusterclustertag) 的数组                | 实例池资源的其他标记。 Azure Databricks 会使用这些标记以及 default_tags 来标记所有的池资源（例如，VM 磁盘卷）。<br><br>Azure Databricks 最多允许 41 个自定义标记。                                                                                                                                                                |
| idle_instance_autotermination_minutes     | `INT32`                                                                | 超出 `min_idle_instances` 的空闲实例在被终止之前由池维护的分钟数。 如果未指定此项，则会在默认的超时期限过后自动终止额外的空闲实例。 如果指定此项，则时间必须在 0 到 10000 分钟之间。 如果提供 0，系统会尽快删除多余的空闲实例。 |
| enable_elastic_disk                       | `BOOL`                                                                 | 自动缩放本地存储：启用后，池中的实例在磁盘空间不足时会动态获取更多磁盘空间。                                                                                                                                                                                                                         |
| disk_spec                                 | [DiskSpec](#clusterdiskspec)                                           | 定义附加到池中每个实例的初始远程存储的容量。                                                                                                                                                                                                                                                                                           |
| preloaded_spark_versions                  | 一个由 `STRING` 构成的数组                                                   | 一个列表，其中包含由池安装在每个实例上的运行时版本。 使用预先加载的运行时版本的池群集启动速度更快，因为不需等待映像下载。 可以通过使用[运行时版本](clusters.md#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。                                    |
| instance_pool_id                          | `STRING`                                                               | 实例池的符合规范的唯一标识符。                                                                                                                                                                                                                                                                                                                        |
| default_tags                              | [ClusterTag](clusters.md#clusterclustertag) 的数组                | 由 Azure Databricks 添加的标记（与任何 custom_tags 无关），其中包括：<br><br>* 供应商：Databricks<br>* DatabricksInstancePoolCreatorId：<create_user_id><br>* DatabricksInstancePoolId：<instance_pool_id>                                                                                                                                                        |
| state                                     | [InstancePoolState](#clusterinstancepoolstate)                         | 实例池的当前状态。                                                                                                                                                                                                                                                                                                                                           |
| stats                                     | [InstancePoolStats](#clusterinstancepoolstats)                         | 有关实例池使用情况的统计信息。                                                                                                                                                                                                                                                                                                                              |

### <a name="azurediskvolumetype"></a><a id="azurediskvolumetype"> </a><a id="clusterazurediskvolumetypepools"> </a>AzureDiskVolumeType

Azure Databricks 支持的所有 Azure 磁盘类型。
请参阅 [https://docs.microsoft.com/azure/virtual-machines/linux/disks-types](/virtual-machines/linux/disks-types)

| 名称             | 描述                                |
|------------------|--------------------------------------------|
| PREMIUM_LRS      | SSD 支持的高级存储层。      |
| STANDARD_LRS     | HDD 支持的标准存储层。     |