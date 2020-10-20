---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/01/2020
title: 群集 API - Azure Databricks
description: 了解 Databricks 群集 API。
ms.openlocfilehash: 292ed1a1c659d42b603d544055cbb1cca1f30793
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937687"
---
# <a name="clusters-api"></a>群集 API

利用群集 API，可以创建、启动、编辑、列出、终止和删除群集。 对群集 API 的请求的最大允许大小为 10MB。

群集生命周期方法需要从[创建](#clusterclusterservicecreatecluster)返回的群集 ID。 若要获取群集列表，请调用[列表](#clusterclusterservicelistclusters)。

Azure Databricks 将群集节点实例类型映射到被称为 DBU 的计算单位。 有关受支持的实例类型及其对应 DBU 的列表，请参阅[实例类型定价页](https://azure.microsoft.com/pricing/details/databricks/)。 有关实例提供程序的信息，请参阅 [Azure 实例类型规范和定价](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)。

Azure Databricks 在停止支持实例类型之前，始终会提供为期一年的弃用通知。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="create"></a><a id="clusterclusterservicecreatecluster"> </a><a id="create"> </a>创建

| 端点                    | HTTP 方法     |
|-----------------------------|-----------------|
| `2.0/clusters/create`       | `POST`          |

创建新的 Apache Spark 群集。 如果有必要，此方法会从云服务提供商处获取新实例。 此方法是异步的；返回的 `cluster_id` 可用于轮询群集状态。 在此方法返回时，群集处于 `PENDING` 状态。
群集在进入 `RUNNING` 状态后即可供使用。 请参阅 [ClusterState](#clusterclusterstate)。

> [!NOTE]
>
> 由于云服务提供商的限制或暂时性的网络问题，Azure Databricks 可能无法获取某些已请求的节点。 如果无法获取足够数量的已请求节点，群集创建将会终止，并显示一条信息性的错误消息。

示例请求：

```json
{
  "cluster_name": "my-cluster",
  "spark_version": "5.3.x-scala2.11",
  "node_type_id": "Standard_D3_v2",
  "spark_conf": {
    "spark.speculation": true
  },
  "num_workers": 25
}
```

下面是一个自动缩放的群集的示例。 此群集最初将包含 `2` 个节点（最小值）。

```json
{
  "cluster_name": "autoscaling-cluster",
  "spark_version": "5.3.x-scala2.11",
  "node_type_id": "Standard_D3_v2",
  "autoscale" : {
    "min_workers": 2,
    "max_workers": 50
  }
}
```

下面是包含要使用的策略的示例。

```json
{
    "num_workers": null,
    "autoscale": {
        "min_workers": 2,
        "max_workers": 8
    },
    "cluster_name": "my-cluster",
    "spark_version": "6.2.x-scala2.11",
    "spark_conf": {},
    "node_type_id": "Standard_D3_v2",
    "custom_tags": {},
    "spark_env_vars": {
        "PYSPARK_PYTHON": "/databricks/python3/bin/python3"
    },
    "autotermination_minutes": 120,
    "init_scripts": [],
    "policy_id": "C65B864F02000008"
}
```

## <a name="create-a-job-using-a-policy-with-the-api"></a>通过该 API 创建使用策略的作业

若要使用策略通过新群集来创建作业或提交运行，请将 `policy_id` 属性添加到请求的 `new_cluster` 规范。

```json
{
  "run_name": "my spark task",
  "new_cluster": {
    "spark_version": "6.0.x-scala2.11",
    "node_type_id": "Standard_D3_v2",
    "num_workers": 10,
    "policy_id": "ABCD000000000000"
  },
  "spark_jar_task": {
    "main_class_name": "com.databricks.ComputeModels"
  }
}
```

### <a name="request-structure"></a><a id="clustercreatecluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称                           | 类型                                                        | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|--------------------------------------|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](#clusterautoscale)                   | 如果是 num_workers，则此项为该群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>注意：在读取群集的属性时，此字段反映的是所需的工作器数，而不是实际的工作器数。 例如，如果将群集的大小从 5 个工作器重设为 10 个工作器，此字段将会立即更新，以反映 10 个工作器的目标大小，而 `executors` 中列出的工作器将会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。                |
| cluster_name                         | `STRING`                                                    | 用户请求的群集名称。 此名称不必唯一。 如果在创建时未指定此字段，群集名称将为空字符串。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| spark_version                        | `STRING`                                                    | 群集的运行时版本。 可以通过使用[运行时版本](#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| spark_conf                           | [SparkConfPair](#clustersparkconfpair)                      | 一个对象，其中包含一组可选的由用户指定的 Spark 配置键值对。 你也可以分别通过以下属性，将额外 JVM 选项的字符串传入到驱动程序和执行程序：<br>`spark.driver.extraJavaOptions` 和 `spark.executor.extraJavaOptions`。<br><br>示例 Spark 配置：<br>`{"spark.speculation": true, "spark.streaming.ui.retainedBatches": 5}` 或<br>`{"spark.driver.extraJavaOptions": "-verbose:gc -XX:+PrintGCDetails"}`                                                                                                                                                                                                                                                          |
| node_type_id                         | `STRING`                                                    | 此字段通过单个值对提供给此群集中的每个 Spark 节点的资源进行编码。 例如，可以针对内存密集型或计算密集型的工作负载来预配和优化 Spark 节点。通过使用[列出节点类型](#clusterclusterservicelistnodetypes) API 调用可以检索可用节点类型的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                         |
| driver_node_type_id                  | `STRING`                                                    | Spark 驱动程序的节点类型。 此字段为可选；如果未设置，驱动程序节点类型将会被设置为与上面定义的 `node_type_id` 相同的值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| custom_tags                          | [ClusterTag](#clusterclustertag)                            | 一个对象，其中包含群集资源的一组标记。 Databricks 会使用这些标记以及 default_tags 来标记所有的群集资源（如 VM）。<br><br>**注意**：<br><br>Azure Databricks 最多允许 43 个自定义标记。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| cluster_log_conf                     | [ClusterLogConf](#clusterclusterlogconf)                    | 用于将 Spark 日志发送到长期存储目标的配置。 对于一个群集，只能指定一个目标。 如果提供该配置，日志将会发送到目标，发送的时间间隔为<br>`5 mins`. 驱动程序日志的目标是 `<destination>/<cluster-ID>/driver`，而执行程序日志的目标是 `<destination>/<cluster-ID>/executor`。                                                                                                                                                                                                                                                                                                                                 |
| init_scripts                         | 一个由 [InitScriptInfo](#clusterclusterinitscriptinfo) 构成的数组 | 用于存储初始化脚本的配置。  可以指定任意数量的脚本。 这些脚本会按照所提供的顺序依次执行。 如果指定了 `cluster_log_conf`，初始化脚本日志将会发送到<br>`<destination>/<cluster-ID>/init_scripts`.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| docker_image                         | [DockerImage](#dockerimage)                                 | [自定义容器](../../../clusters/custom-containers.md)的 Docker 映像。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| spark_env_vars                       | [SparkEnvPair](#clustersparkenvpair)                        | 一个对象，其中包含一组可选的由用户指定的环境变量键值对。 在启动驱动程序和工作器时，(X,Y) 形式的键值对会按原样导出（即<br>`export X='Y'`）。<br><br>若要额外指定一组 `SPARK_DAEMON_JAVA_OPTS`，建议将其追加到 `$SPARK_DAEMON_JAVA_OPTS`，如以下示例中所示。 这样就确保了还会包含所有默认的 databricks 托管环境变量。<br><br>Spark 环境变量示例：<br>`{"SPARK_WORKER_MEMORY": "28000m", "SPARK_LOCAL_DIRS": "/local_disk0"}` 或<br>`{"SPARK_DAEMON_JAVA_OPTS": "$SPARK_DAEMON_JAVA_OPTS -Dspark.shuffle.service.enabled=true"}` |
| autotermination_minutes              | `INT32`                                                     | 在群集处于不活动状态的时间达到此时间（以分钟为单位）后，自动终止该群集。 如果未设置，将不会自动终止此群集。 如果指定，阈值必须介于 10 到 10000 分钟之间。 你也可以将此值设置为 0，以显式禁用自动终止。                                                                                                                                                                                                                                                                                                                                                                                                                    |
| instance_pool_id                     | `STRING`                                                    | 群集所属的实例池的可选 ID。 有关详细信息，请参阅[实例池 API](instance-pools.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| idempotency_token                    | `STRING`                                                    | 可选令牌，可用于保证群集创建请求的幂等性。 如果已经存在具有提供的令牌的活动群集，该请求将不会创建新群集，而是会返回现有群集的 ID。 不会针对已终止的群集检查是否存在具有相同令牌的群集。<br><br>如果指定幂等性令牌，则可在失败时重试，直到该请求成功。 Azure Databricks 将会确保只有一个群集将通过该幂等性令牌启动。<br><br>此令牌最多只能包含 64 个字符。                                                                                           |

### <a name="response-structure"></a><a id="clustercreateclusterresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型           | 描述                               |
|----------------|----------------|-------------------------------------------|
| cluster_id     | `STRING`       | 该群集的规范标识符。     |

## <a name="edit"></a><a id="clusterclusterserviceeditcluster"> </a><a id="edit"> </a>编辑

| 端点                  | HTTP 方法     |
|---------------------------|-----------------|
| `2.0/clusters/edit`       | `POST`          |

编辑群集的配置，以匹配所提供的属性和大小。

如果群集处于 `RUNNING` 或 `TERMINATED` 状态，则可编辑该群集。
如果在群集处于 `RUNNING` 状态时编辑该群集，它将会重启，以便使新属性能够生效。 如果在群集处于 `TERMINATED` 状态时编辑该群集，它将保持 `TERMINATED` 状态。 在下次使用 `clusters/start` API 启动它时，新属性将会生效。 尝试编辑处于任何其他状态的群集将会被拒绝，并返回 `INVALID_STATE` 错误代码。

无法编辑由 Databricks 作业服务创建的群集。

示例请求：

```json
{
 "cluster_id": "1202-211320-brick1",
 "num_workers": 10,
 "spark_version": "5.3.x-scala2.11",
 "node_type_id": "Standard_D3_v2"
}
```

### <a name="request-structure"></a><a id="clustereditcluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称                           | 类型                                                        | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|--------------------------------------|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](#clusterautoscale)                   | 如果是 num_workers，则此项为该群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>注意：在读取群集的属性时，此字段反映的是所需的工作器数，而不是实际的工作器数。 例如，如果将群集的大小从 5 个工作器重设为 10 个工作器，此字段将会立即更新，以反映 10 个工作器的目标大小，而 `executors` 中列出的工作器将会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。                |
| cluster_id                           | `STRING`                                                    | 该群集的规范标识符。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cluster_name                         | `STRING`                                                    | 用户请求的群集名称。 此名称不必唯一。 如果在创建时未指定此字段，群集名称将为空字符串。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| spark_version                        | `STRING`                                                    | 群集的运行时版本。 可以通过使用[运行时版本](#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| spark_conf                           | [SparkConfPair](#clustersparkconfpair)                      | 一个对象，其中包含一组可选的由用户指定的 Spark 配置键值对。 你也可以分别通过以下属性，将额外 JVM 选项的字符串传入到驱动程序和执行程序：<br>`spark.driver.extraJavaOptions` 和 `spark.executor.extraJavaOptions`。<br><br>示例 Spark 配置：<br>`{"spark.speculation": true, "spark.streaming.ui.retainedBatches": 5}` 或<br>`{"spark.driver.extraJavaOptions": "-verbose:gc -XX:+PrintGCDetails"}`                                                                                                                                                                                                                                                          |
| node_type_id                         | `STRING`                                                    | 此字段通过单个值对提供给此群集中的每个 Spark 节点的资源进行编码。 例如，可以针对内存密集型或计算密集型的工作负载来预配和优化 Spark 节点。通过使用[列出节点类型](#clusterclusterservicelistnodetypes) API 调用可以检索可用节点类型的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                         |
| driver_node_type_id                  | `STRING`                                                    | Spark 驱动程序的节点类型。 此字段为可选；如果未设置，驱动程序节点类型将会被设置为与上面定义的 `node_type_id` 相同的值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| cluster_log_conf                     | [ClusterLogConf](#clusterclusterlogconf)                    | 用于将 Spark 日志发送到长期存储目标的配置。 对于一个群集，只能指定一个目标。 如果提供该配置，日志将会发送到目标，发送的时间间隔为<br>`5 mins`. 驱动程序日志的目标是 `<destination>/<cluster-ID>/driver`，而执行程序日志的目标是 `<destination>/<cluster-ID>/executor`。                                                                                                                                                                                                                                                                                                                                 |
| init_scripts                         | 一个由 [InitScriptInfo](#clusterclusterinitscriptinfo) 构成的数组 | 用于存储初始化脚本的配置。  可以指定任意数量的目标。 这些脚本会按照所提供的顺序依次执行。 如果指定了 `cluster_log_conf`，初始化脚本日志将会发送到<br>`<destination>/<cluster-ID>/init_scripts`.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| docker_image                         | [DockerImage](#dockerimage)                                 | [自定义容器](../../../clusters/custom-containers.md)的 Docker 映像。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| spark_env_vars                       | [SparkEnvPair](#clustersparkenvpair)                        | 一个对象，其中包含一组可选的由用户指定的环境变量键值对。 在启动驱动程序和工作器时，(X,Y) 形式的键值对会按原样导出（即<br>`export X='Y'`）。<br><br>若要额外指定一组 `SPARK_DAEMON_JAVA_OPTS`，建议将其追加到 `$SPARK_DAEMON_JAVA_OPTS`，如以下示例中所示。 这样就确保了还会包含所有默认的 databricks 托管环境变量。<br><br>Spark 环境变量示例：<br>`{"SPARK_WORKER_MEMORY": "28000m", "SPARK_LOCAL_DIRS": "/local_disk0"}` 或<br>`{"SPARK_DAEMON_JAVA_OPTS": "$SPARK_DAEMON_JAVA_OPTS -Dspark.shuffle.service.enabled=true"}` |
| autotermination_minutes              | `INT32`                                                     | 在群集处于不活动状态的时间达到此时间（以分钟为单位）后，自动终止该群集。 如果未设置，将不会自动终止此群集。 如果指定，阈值必须介于 10 到 10000 分钟之间。 你也可以将此值设置为 0，以显式禁用自动终止。                                                                                                                                                                                                                                                                                                                                                                                                                    |
| instance_pool_id                     | `STRING`                                                    | 群集所属的实例池的可选 ID。 有关详细信息，请参阅[池](../../../clusters/instance-pools/index.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

## <a name="start"></a><a id="clusterclusterservicestartcluster"> </a><a id="start"> </a>启动

| 端点                   | HTTP 方法     |
|----------------------------|-----------------|
| `2.0/clusters/start`       | `POST`          |

启动已终止的群集（如果提供了该群集的 ID）。 这类似于 `createCluster`，区别在于：

* 已终止的群集 ID 和属性会被保留。
* 该群集启动时的大小为最后指定的群集大小。 如果已终止的群集是自动缩放的群集，该群集启动时只包含最少数量的节点。
* 如果该群集处于 `RESTARTING` 状态，则会返回 `400` 错误。
* 你无法启动为运行作业而启动的群集。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

### <a name="request-structure"></a><a id="clusterstartcluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                        |
|----------------|----------------|----------------------------------------------------|
| cluster_id     | `STRING`       | 要启动的群集。 此字段为必需字段。 |

## <a name="restart"></a><a id="clusterclusterservicerestartcluster"> </a><a id="restart"> </a>重启

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/clusters/restart`       | `POST`          |

如果有群集的 ID，则重启该群集。 该群集必须处于 `RUNNING` 状态。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

### <a name="request-structure"></a><a id="clusterrestartcluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                        |
|----------------|----------------|----------------------------------------------------|
| cluster_id     | `STRING`       | 要启动的群集。 此字段为必需字段。 |

## <a name="resize"></a><a id="clusterclusterserviceresizecluster"> </a><a id="resize"> </a>重设大小

| 端点                    | HTTP 方法     |
|-----------------------------|-----------------|
| `2.0/clusters/resize`       | `POST`          |

重设群集大小，使其具有所需的工作器数。 该群集必须处于 `RUNNING` 状态。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1",
  "num_workers": 30
}
```

### <a name="request-structure"></a><a id="clusterresizecluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称                           | 类型                                      | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|--------------------------------------|-------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](#clusterautoscale) | 如果是 num_workers，则此项为该群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>**注意：** 在读取群集的属性时，此字段反映的是所需的工作器数，而不是实际的工作器数。 例如，如果将群集的大小从 5 个工作器重设为 10 个工作器，此字段将会立即更新，以反映 10 个工作器的目标大小，而 `executors` 中列出的工作器将会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。 |
| cluster_id                           | `STRING`                                  | 要重设大小的群集。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

## <a name="delete-terminate"></a><a id="clusterclusterservicedeletecluster"> </a><a id="delete-terminate"> </a>删除（终止）

| 端点                    | HTTP 方法     |
|-----------------------------|-----------------|
| `2.0/clusters/delete`       | `POST`          |

终止群集（如果提供了该群集的 ID）。 该群集会以异步方式被删除。 终止完成之后，该群集将会处于 `TERMINATED` 状态。 如果该群集已处于 `TERMINATING` 或 `TERMINATED` 状态，则不会执行任何操作。

群集在终止 30 天后会[永久删除](#clusterclusterservicepermanentdeletecluster)。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

### <a name="request-structure"></a><a id="clusterdeletecluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                           |
|----------------|----------------|-------------------------------------------------------|
| cluster_id     | `STRING`       | 要终止的群集。 此字段为必需字段。 |

## <a name="permanent-delete"></a><a id="clusterclusterservicepermanentdeletecluster"> </a><a id="permanent-delete"> </a>永久删除

| 端点                              | HTTP 方法     |
|---------------------------------------|-----------------|
| `2.0/clusters/permanent-delete`       | `POST`          |

永久删除群集。 如果该群集正在运行，则它会被终止，并且其资源会以异步方式删除。 如果该群集已终止，则它会被立即删除。

对于永久删除的群集，无法执行任何操作，包括检索该群集的权限。 群集列表中也不会再返回已永久删除的群集。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

### <a name="request-structure"></a><a id="clusterpermanentdeletecluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                    |
|----------------|----------------|----------------------------------------------------------------|
| cluster_id     | `STRING`       | 要永久删除的群集。 此字段为必需字段。 |

## <a name="get"></a><a id="clusterclusterservicegetcluster"> </a><a id="get"> </a>获取

| 端点                 | HTTP 方法     |
|--------------------------|-----------------|
| `2.0/clusters/get`       | `GET`           |

检索群集的信息（如果提供了该群集的标识符）。
当群集正在运行时，或在群集被终止之后的 30 天之内，可以描述群集。

示例请求：

```bash
/clusters/get?cluster_id=1202-211320-brick1
```

### <a name="request-structure"></a><a id="clustergetcluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                              |
|----------------|----------------|--------------------------------------------------------------------------|
| cluster_id     | `STRING`       | 要检索其信息的群集。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="clustergetclusterresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称                           | 类型                                                            | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|--------------------------------------|-----------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](#clusterautoscale)                       | 如果是 num_workers，则此项为该群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>注意：在读取群集的属性时，此字段反映的是所需的工作器数，而不是实际的工作器数。 例如，如果将群集的大小从 5 个工作器重设为 10 个工作器，此字段将会立即更新，以反映 10 个工作器的目标大小，而 `executors` 中列出的工作器将会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。                |
| cluster_id                           | `STRING`                                                        | 该群集的规范标识符。 此 ID 在群集重启和重设大小期间保留，同时每个新群集都有一个全局唯一的 ID。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| creator_user_name                    | `STRING`                                                        | 创建者用户名。 如果已删除该用户，响应中将不会包括此字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| 驱动程序                               | [SparkNode](#clustersparkinfosparknode)                         | Spark 驱动程序所驻留的节点。 该驱动程序节点包含 Spark Master 和 Databricks 应用程序，该应用程序管理每个笔记本的 Spark REPL。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| 执行程序                            | 一个由 [SparkNode](#clustersparkinfosparknode) 构成的数组             | Spark 执行程序所驻留的节点。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| spark_context_id                     | `INT64`                                                         | 规范的 SparkContext 标识符。 此值在 Spark 驱动程序重启时会更改。 `(cluster_id, spark_context_id)` 对是所有 Spark 上下文中的全局唯一标识符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| jdbc_port                            | `INT32`                                                         | Spark JDBC 服务器用于在驱动程序节点中进行侦听的端口。 在执行程序节点中，没有服务会在此端口上进行侦听。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| cluster_name                         | `STRING`                                                        | 用户请求的群集名称。 此名称不必唯一。 如果在创建时未指定此字段，群集名称将为空字符串。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| spark_version                        | `STRING`                                                        | 群集的运行时版本。 可以通过使用[运行时版本](#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| spark_conf                           | [SparkConfPair](#clustersparkconfpair)                          | 一个对象，其中包含一组可选的由用户指定的 Spark 配置键值对。 你也可以分别通过以下属性，将额外 JVM 选项的字符串传入到驱动程序和执行程序：<br>`spark.driver.extraJavaOptions` 和 `spark.executor.extraJavaOptions`。<br><br>示例 Spark 配置：<br>`{"spark.speculation": true, "spark.streaming.ui.retainedBatches": 5}` 或<br>`{"spark.driver.extraJavaOptions": "-verbose:gc -XX:+PrintGCDetails"}`                                                                                                                                                                                                                                                          |
| node_type_id                         | `STRING`                                                        | 此字段通过单个值对提供给此群集中的每个 Spark 节点的资源进行编码。 例如，可以针对内存密集型或计算密集型的工作负载来预配和优化 Spark 节点。通过使用[列出节点类型](#clusterclusterservicelistnodetypes) API 调用可以检索可用节点类型的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                         |
| driver_node_type_id                  | `STRING`                                                        | Spark 驱动程序的节点类型。 此字段为可选；如果未设置，驱动程序节点类型将会被设置为与上面定义的 `node_type_id` 相同的值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| custom_tags                          | [ClusterTag](#clusterclustertag)                                | 一个对象，其中包含群集资源的一组标记。 Databricks 会使用这些标记以及 default_tags 来标记所有的群集资源。<br><br>**注意**：<br><br>* 旧版节点类型（如计算优化和内存优化）不支持标记<br>* Databricks 最多允许 45 个自定义标记                                                                                                                                                                                                                                                                                                                                                                                                       |
| cluster_log_conf                     | [ClusterLogConf](#clusterclusterlogconf)                        | 用于将 Spark 日志发送到长期存储目标的配置。 对于一个群集，只能指定一个目标。 如果提供该配置，日志将会发送到目标，发送的时间间隔为<br>`5 mins`. 驱动程序日志的目标是 `<destination>/<cluster-ID>/driver`，而执行程序日志的目标是 `<destination>/<cluster-ID>/executor`。                                                                                                                                                                                                                                                                                                                                 |
| init_scripts                         | 一个由 [InitScriptInfo](#clusterclusterinitscriptinfo) 构成的数组     | 用于存储初始化脚本的配置。  可以指定任意数量的目标。 这些脚本会按照所提供的顺序依次执行。 如果指定了 `cluster_log_conf`，初始化脚本日志将会发送到<br>`<destination>/<cluster-ID>/init_scripts`.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| docker_image                         | [DockerImage](#dockerimage)                                     | [自定义容器](../../../clusters/custom-containers.md)的 Docker 映像。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| spark_env_vars                       | [SparkEnvPair](#clustersparkenvpair)                            | 一个对象，其中包含一组可选的由用户指定的环境变量键值对。 在启动驱动程序和工作器时，(X,Y) 形式的键值对会按原样导出（即<br>`export X='Y'`）。<br><br>若要额外指定一组 `SPARK_DAEMON_JAVA_OPTS`，建议将其追加到 `$SPARK_DAEMON_JAVA_OPTS`，如以下示例中所示。 这样就确保了还会包含所有默认的 databricks 托管环境变量。<br><br>Spark 环境变量示例：<br>`{"SPARK_WORKER_MEMORY": "28000m", "SPARK_LOCAL_DIRS": "/local_disk0"}` 或<br>`{"SPARK_DAEMON_JAVA_OPTS": "$SPARK_DAEMON_JAVA_OPTS -Dspark.shuffle.service.enabled=true"}` |
| autotermination_minutes              | `INT32`                                                         | 在群集处于不活动状态的时间达到此时间（以分钟为单位）后，自动终止该群集。 如果未设置，将不会自动终止此群集。 如果指定，阈值必须介于 10 到 10000 分钟之间。 你也可以将此值设置为 0，以显式禁用自动终止。                                                                                                                                                                                                                                                                                                                                                                                                                    |
| enable_elastic_disk                  | `BOOL`                                                          | 自动缩放本地存储：启用后，此群集在其 Spark 工作器磁盘空间不足时将会动态获取更多磁盘空间。 有关详细信息，请参阅[自动缩放本地存储](../../../clusters/configure.md#autoscaling-local-storage-azure)。                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| instance_pool_id                     | `STRING`                                                        | 群集所属的实例池的可选 ID。 有关详细信息，请参阅[池](../../../clusters/instance-pools/index.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| state                                | [ClusterState](#clusterclusterstate)                            | 群集的状态。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| state_message                        | `STRING`                                                        | 与最新状态转换关联的消息（例如，群集进入 `TERMINATED` 状态的原因）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| start_time                           | `INT64`                                                         | （在群集进入 `PENDING` 状态时）收到群集创建请求的时间（以 epoch 毫秒表示）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| terminated_time                      | `INT64`                                                         | 终止群集（如果适用）的时间，以 epoch 毫秒表示。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| last_state_loss_time                 | `INT64`                                                         | 群集驱动程序上一次丢失其状态（由于重启或驱动程序故障）的时间。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| last_activity_time                   | `INT64`                                                         | 群集上次处于活动状态的时间（以 epoch 毫秒表示）。 如果群集上至少有一个命令尚未完成，则该群集处于活动状态。 此字段在群集已到达 `RUNNING` 状态之后可用。 我们会尽最大努力尝试对此字段进行更新。 某些版本的 Spark 不支持报告群集活动性。 有关详细信息，请参阅[自动终止](../../../clusters/clusters-manage.md#automatic-termination)。                                                                                                                                                                                                                                        |
| cluster_memory_mb                    | `INT64`                                                         | 以兆字节表示的群集内存总量。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cluster_cores                        | `FLOAT`                                                         | 可用于此群集的 CPU 内核数。 这可以是小数，因为某些节点类型被配置为在同一实例上的 Spark 节点之间共享内核。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| default_tags                         | [ClusterTag](#clusterclustertag)                                | 一个对象，其中包含一组由 Azure Databricks 添加（不管任何 custom_tags 如何）的标记，包括：<br><br>* Vendor:Databricks<br>* Creator: <username-of-creator><br>* ClusterName: <name-of-cluster><br>* ClusterId: <id-of-cluster><br>* Name:<Databricks internal use>      在作业群集上：<br>  <br>* RunName: <name-of-job><br>* JobId: <id-of-job>                                                                                                                                                                                                                                                                                                                                              |
| cluster_log_status                   | [LogSyncStatus](#clusterlogsyncstatus)                          | 群集日志发送状态。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| termination_reason                   | [TerminationReason](#clusterterminationreasonterminationreason) | 有关群集终止原因的信息。 此字段只在群集处于 `TERMINATING` 或 `TERMINATED` 状态时才会显示。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## <a name="pin"></a><a id="clusterclusterservicepincluster"> </a><a id="pin"> </a>固定

> [!NOTE]
>
> 只有 Azure Databricks [管理员](../../../administration-guide/users-groups/users.md)才能调用此 API。

| 端点                 | HTTP 方法     |
|--------------------------|-----------------|
| `2.0/clusters/pin`       | `POST`          |

确保即使在群集已被终止的时间超过了 30 天的情况下仍会保留通用群集配置。
“固定”可确保群集始终会由[列表](#clusterclusterservicelistclusters) API 返回。
固定一个已经固定的群集不会起作用。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

### <a name="request-structure"></a><a id="clusterpincluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                          |
|----------------|----------------|------------------------------------------------------|
| cluster_id     | `STRING`       | 要固定的群集。 此字段为必需字段。          |

## <a name="unpin"></a><a id="clusterclusterserviceunpincluster"> </a><a id="unpin"> </a>取消固定

> [!NOTE]
>
> 只有 Azure Databricks [管理员](../../../administration-guide/users-groups/users.md)才能调用此 API。

| 端点                   | HTTP 方法     |
|----------------------------|-----------------|
| `2.0/clusters/unpin`       | `POST`          |

允许最终从[列表](#clusterclusterservicelistclusters) API 返回的列表中删除群集。
对一个未固定的群集取消固定不会起作用。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

### <a name="request-structure"></a><a id="clusterunpincluster"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                          |
|----------------|----------------|------------------------------------------------------|
| cluster_id     | `STRING`       | 要取消固定的群集。 此字段为必需字段。        |

## <a name="list"></a><a id="clusterclusterservicelistclusters"> </a><a id="list"> </a>列表

| 端点                  | HTTP 方法     |
|---------------------------|-----------------|
| `2.0/clusters/list`       | `GET`           |

返回有关所有固定群集、活动群集、过去 30 天内最多 70 个最近终止的通用群集以及过去 30 天内最多 30 个最近终止的作业群集的相关信息。 例如，如果有 1 个固定群集、4 个活动群集、45 个在过去 30 天内终止的通用群集以及 50 个在过去 30 天内终止的作业群集，则此 API 会返回 1 个固定群集、4 个活动群集、所有 45 个终止的通用群集，以及 30 个最近终止的作业群集。

### <a name="response-structure"></a><a id="clusterlistclustersresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型                                           | 描述                               |
|----------------|------------------------------------------------|-------------------------------------------|
| clusters       | 一个由 [ClusterInfo](#clusterclusterinfo) 构成的数组 | 群集的列表。                       |

## <a name="list-node-types"></a><a id="clusterclusterservicelistnodetypes"> </a><a id="list-node-types"> </a>列出节点类型

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/clusters/list-node-types`       | `GET`           |

返回受支持的 Spark 节点类型的列表。 这些节点类型可用于启动群集。

### <a name="response-structure"></a><a id="clusterlistnodetypesresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称               | 类型                                     | 描述                               |
|--------------------------|------------------------------------------|-------------------------------------------|
| node_types               | 一个由 [NodeType](#clusternodetype) 构成的数组 | 可用的 Spark 节点类型的列表。   |

## <a name="runtime-versions"></a><a id="clusterclusterservicelistsparkversions"> </a><a id="runtime-versions"> </a>运行时版本

| 端点                            | HTTP 方法     |
|-------------------------------------|-----------------|
| `2.0/clusters/spark-versions`       | `GET`           |

返回可用的[运行时版本](#clustersparkversion)的列表。 这些版本可用于启动群集。

### <a name="response-structure"></a><a id="clustergetsparkversionsresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称              | 类型                                             | 描述                          |
|-------------------------|--------------------------------------------------|--------------------------------------|
| versions                | 一个由 [SparkVersion](#clustersparkversion) 构成的数组 | 所有可用的运行时版本。  |

## <a name="events"></a><a id="clusterclusterservicegetevents"> </a><a id="events"> </a>事件

| 端点                    | HTTP 方法     |
|-----------------------------|-----------------|
| `2.0/clusters/events`       | `POST`          |

检索有关群集活动性的事件列表。 你可以检索活动的（正在运行的、挂起的或正在重新配置的）群集和已终止的群集（自其上次终止后 30 天内）中的事件。
此 API 支持分页。 如果有更多事件需要读取，响应会包括用于请求下一页事件所需的所有参数。

示例请求：

```json
{
  "cluster_id": "1202-211320-brick1"
}
```

示例响应：

```json
{
  "events": [{
    "cluster_id": "1202-211320-brick1",
    "timestamp": 1534371918659,
    "type": "TERMINATING",
    "details": {
      "reason": {
        "code": "INACTIVITY",
        "parameters": {
          "inactivity_duration_min": "120"
        }
      }
    }
  }, {
    "cluster_id": "1202-211320-brick1",
    "timestamp": 1534358289590,
    "type": "RUNNING",
    "details": {
      "current_num_workers": 2,
      "target_num_workers": 2
    }
  }, {
    "cluster_id": "1202-211320-brick1",
    "timestamp": 1533225298406,
    "type": "RESTARTING",
    "details": {
      "user": "admin"
    }
  }],
  "next_page": {
    "cluster_id": "0802-034608-aloe926",
    "end_time": 1534371918659,
    "offset": 50
  },
  "total_count": 55
}
```

用于检索下一页事件的请求示例：

```json
{
  "cluster_id": "1202-211320",
  "start_time": 1534371918659
}
```

### <a name="request-structure"></a><a id="clustergetevents"> </a><a id="request-structure"> </a>请求结构

检索与某个特定群集相关的事件。

| 字段名称      | 类型                                                           | 描述                                                                                                                                                               |
|-----------------|----------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cluster_id      | `STRING`                                                       | 要检索其相关事件的群集的 ID。 此字段为必需字段。                                                                                                   |
| start_time      | `INT64`                                                        | 以 epoch 毫秒表示的开始时间。 如果为空，则返回从起始时间开始的事件。                                                                       |
| end_time        | `INT64`                                                        | 以 epoch 毫秒表示的结束时间。 如果为空，则返回截止到当前时间的事件。                                                                                      |
| 顺序           | [ListOrder](#clusterlistorder)                                 | 列出事件时采用的顺序；`ASC` 或 `DESC`。 默认为 `DESC`。                                                                                                  |
| event_types     | 一个由 [ClusterEventType](#clustereventsclustereventtype) 构成的数组 | 可选的一组要筛选的事件类型。 如果为空，则返回所有事件类型。                                                                                      |
| offset          | `INT64`                                                        | 结果集之中的偏移量。 默认为 0（无偏移）。 如果指定了偏移量，并按降序请求结果，则“end_time”字段为必填字段。 |
| limit           | `INT64`                                                        | 要包括在一页事件中的最大事件数。 默认为 50，允许的最大值为 500。                                                            |

### <a name="response-structure"></a><a id="clustergeteventsresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称      | 类型                                                   | 说明                                                                                               |
|-----------------|--------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| 活动          | 一个由 [ClusterEvent](#clustereventsclusterevent) 构成的数组 | 匹配事件的列表。                                                                             |
| next_page       | [请求结构](#clustergetevents)                 | 检索下一页事件所需的参数。 如果没有更多要读取的事件，则省略此字段。 |
| total_count     | `INT64`                                                | 按 Start_time、end_time 和 event_types 筛选的事件的总数。                         |

## <a name="data-structures"></a><a id="clusteradd"> </a><a id="data-structures"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [自动缩放](#autoscale)
* [ClusterInfo](#clusterinfo)
* [ClusterEvent](#clusterevent)
* [ClusterEventType](#clustereventtype)
* [EventDetails](#eventdetails)
* [ClusterAttributes](#clusterattributes)
* [clusterSize](#clustersize)
* [ListOrder](#listorder)
* [ResizeCause](#resizecause)
* [ClusterLogConf](#clusterlogconf)
* [InitScriptInfo](#initscriptinfo)
* [ClusterTag](#clustertag)
* [DbfsStorageInfo](#dbfsstorageinfo)
* [FileStorageInfo](#filestorageinfo)
* [DockerImage](#dockerimage)
* [DockerBasicAuth](#dockerbasicauth)
* [LogSyncStatus](#logsyncstatus)
* [NodeType](#nodetype)
* [ClusterCloudProviderNodeInfo](#clustercloudprovidernodeinfo)
* [ClusterCloudProviderNodeStatus](#clustercloudprovidernodestatus)
* [ParameterPair](#parameterpair)
* [SparkConfPair](#sparkconfpair)
* [SparkEnvPair](#sparkenvpair)
* [SparkNode](#sparknode)
* [SparkVersion](#sparkversion)
* [TerminationReason](#terminationreason)
* [PoolClusterTerminationCode](#poolclusterterminationcode)
* [ClusterSource](#clustersource)
* [ClusterState](#clusterstate)
* [TerminationCode](#terminationcode)
* [TerminationType](#terminationtype)
* [TerminationParameter](#terminationparameter)

### <a name="autoscale"></a><a id="autoscale"> </a><a id="clusterautoscale"> </a>AutoScale

定义群集工作器的最小数量和最大数量的范围。

| 字段名称      | 类型          | 描述                                                                                                                                                          |
|-----------------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| min_workers     | `INT32`       | 群集在未充分利用时可纵向缩减到的最小工作器数。 此数量也是群集在创建后将会具有的初始工作器的数量。 |
| max_workers     | `INT32`       | 群集在负载过高时可纵向扩展到的最大工作器数。 max_workers 必须严格大于 min_workers。                              |

### <a name="clusterinfo"></a><a id="clusterclusterinfo"> </a><a id="clusterinfo"> </a>ClusterInfo

有关群集的元数据。

| 字段名称                           | 类型                                                            | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------------|-----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](#clusterautoscale)                       | 如果是 num_workers，则此项为该群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>注意：在读取群集的属性时，此字段反映的是所需的工作器数，而不是实际的工作器数。 例如，如果将群集的大小从 5 个工作器重设为 10 个工作器，此字段将会立即更新，以反映 10 个工作器的目标大小，而 `executors` 中列出的工作器将会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。       |
| cluster_id                           | `STRING`                                                        | 该群集的规范标识符。 此 ID 在群集重启和重设大小期间保留，同时每个新群集都有一个全局唯一的 ID。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| creator_user_name                    | `STRING`                                                        | 创建者用户名。 如果已删除该用户，响应中将不会包括此字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 驱动程序                               | [SparkNode](#clustersparkinfosparknode)                         | Spark 驱动程序所驻留的节点。 该驱动程序节点包含 Spark Master 和 Databricks 应用程序，该应用程序管理每个笔记本的 Spark REPL。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| 执行程序                            | 一个由 [SparkNode](#clustersparkinfosparknode) 构成的数组             | Spark 执行程序所驻留的节点。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| spark_context_id                     | `INT64`                                                         | 规范的 SparkContext 标识符。 此值在 Spark 驱动程序重启时会更改。 `(cluster_id, spark_context_id)` 对是所有 Spark 上下文中的全局唯一标识符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| jdbc_port                            | `INT32`                                                         | Spark JDBC 服务器用于在驱动程序节点中进行侦听的端口。 在执行程序节点中，没有服务会在此端口上进行侦听。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| cluster_name                         | `STRING`                                                        | 用户请求的群集名称。 此名称不必唯一。 如果在创建时未指定此字段，群集名称将为空字符串。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| spark_version                        | `STRING`                                                        | 群集的运行时版本。 可以通过使用[运行时版本](#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| spark_conf                           | [SparkConfPair](#clustersparkconfpair)                          | 一个对象，其中包含一组可选的由用户指定的 Spark 配置键值对。 你也可以分别通过以下属性，将额外 JVM 选项的字符串传入到驱动程序和执行程序：<br>`spark.driver.extraJavaOptions` 和 `spark.executor.extraJavaOptions`。<br><br>示例 Spark 配置：<br>`{"spark.speculation": true, "spark.streaming.ui.retainedBatches": 5}` 或<br>`{"spark.driver.extraJavaOptions": "-verbose:gc -XX:+PrintGCDetails"}`                                                                                                                                                                                                                                                 |
| node_type_id                         | `STRING`                                                        | 此字段通过单个值对提供给此群集中的每个 Spark 节点的资源进行编码。 例如，可以针对内存或计算密集型工作负载对 Spark 节点进行预配和优化。 通过使用[列出节点类型](#clusterclusterservicelistnodetypes) API 调用，可以检索可用节点类型的列表。                                                                                                                                                                                                                                                                                                                                                       |
| driver_node_type_id                  | `STRING`                                                        | Spark 驱动程序的节点类型。 此字段为可选；如果未设置，驱动程序节点类型将会被设置为与上面定义的 `node_type_id` 相同的值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| cluster_log_conf                     | [ClusterLogConf](#clusterclusterlogconf)                        | 用于将 Spark 日志发送到长期存储目标的配置。 对于一个群集，只能指定一个目标。 如果提供该配置，日志将会发送到目标，发送的时间间隔为<br>`5 mins`. 驱动程序日志的目标是 `<destination>/<cluster-ID>/driver`，而执行程序日志的目标是 `<destination>/<cluster-ID>/executor`。                                                                                                                                                                                                                                                                                                                        |
| init_scripts                         | 一个由 [InitScriptInfo](#clusterclusterinitscriptinfo) 构成的数组     | 用于存储初始化脚本的配置。  可以指定任意数量的目标。 这些脚本会按照所提供的顺序依次执行。 如果指定了 `cluster_log_conf`，初始化脚本日志将会发送到<br>`<destination>/<cluster-ID>/init_scripts`.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| docker_image                         | [DockerImage](#dockerimage)                                     | [自定义容器](../../../clusters/custom-containers.md)的 Docker 映像。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| spark_env_vars                       | [SparkEnvPair](#clustersparkenvpair)                            | 一个对象，其中包含一组可选的由用户指定的环境变量键值对。 在启动驱动程序和工作器时，(X,Y) 形式的键值对会按原样导出（即<br>`export X='Y'`）。<br><br>若要额外指定一组 `SPARK_DAEMON_JAVA_OPTS`，建议将其追加到 `$SPARK_DAEMON_JAVA_OPTS`，如以下示例中所示。 这样就确保了还会包含所有默认的 databricks 托管环境变量。<br><br>Spark 环境变量示例：<br>`{"SPARK_WORKER_MEMORY": "28000m", "SPARK_LOCAL_DIRS": "/local_disk0"}` 或<br>`{"SPARK_DAEMON_JAVA_OPTS": "$SPARK_DAEMON_JAVA_OPTS -Dspark.shuffle.service.enabled=true"}` |
| autotermination_minutes              | `INT32`                                                         | 在群集处于不活动状态的时间达到此时间（以分钟为单位）后，自动终止该群集。 如果未设置，将不会自动终止此群集。 如果指定，阈值必须介于 10 到 10000 分钟之间。 你也可以将此值设置为 0，以显式禁用自动终止。                                                                                                                                                                                                                                                                                                                                                                                                           |
| enable_elastic_disk                  | `BOOL`                                                          | 自动缩放本地存储：启用后，此群集在其 Spark 工作器磁盘空间不足时将会动态获取更多磁盘空间。 有关详细信息，请参阅[自动缩放本地存储](../../../clusters/configure.md#autoscaling-local-storage-azure)。                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| instance_pool_id                     | `STRING`                                                        | 群集所属的实例池的可选 ID。 有关详细信息，请参阅[池](../../../clusters/instance-pools/index.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| state                                | [ClusterState](#clusterclusterstate)                            | 群集的状态。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| state_message                        | `STRING`                                                        | 与最近的状态转换关联的消息（例如，群集进入 `TERMINATED` 状态的原因）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| start_time                           | `INT64`                                                         | （在群集进入 `PENDING` 状态时）收到群集创建请求的时间（以 epoch 毫秒表示）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| terminated_time                      | `INT64`                                                         | 终止群集（如果适用）的时间，以 epoch 毫秒表示。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| last_state_loss_time                 | `INT64`                                                         | 群集驱动程序上一次丢失其状态（由于重启或驱动程序故障）的时间。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| last_activity_time                   | `INT64`                                                         | 群集上次处于活动状态的时间（以 epoch 毫秒表示）。 如果群集上至少有一个命令尚未完成，则该群集处于活动状态。 此字段在群集已到达 `RUNNING` 状态之后可用。 我们会尽最大努力尝试对此字段进行更新。 某些版本的 Spark 不支持报告群集活动性。 有关详细信息，请参阅[自动终止](../../../clusters/clusters-manage.md#automatic-termination)。                                                                                                                                                                                                                                 |
| cluster_memory_mb                    | `INT64`                                                         | 以兆字节表示的群集内存总量。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| cluster_cores                        | `FLOAT`                                                         | 可用于此群集的 CPU 内核数。 这可以是小数，因为某些节点类型被配置为在同一实例上的 Spark 节点之间共享内核。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| default_tags                         | [ClusterTag](#clusterclustertag)                                | 一个对象，其中包含一组由 Azure Databricks 添加（不管任何 custom_tags 如何）的标记，包括：<br><br>* Vendor:Databricks<br>* Creator: <username-of-creator><br>* ClusterName: <name-of-cluster><br>* ClusterId: <id-of-cluster><br>* Name:<Databricks internal use>      在作业群集上：<br>  <br>* RunName: <name-of-job><br>* JobId: <id-of-job>                                                                                                                                                                                                                                                                                                                                     |
| cluster_log_status                   | [LogSyncStatus](#clusterlogsyncstatus)                          | 群集日志发送状态。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| termination_reason                   | [TerminationReason](#clusterterminationreasonterminationreason) | 有关群集终止原因的信息。 此字段只在群集处于 `TERMINATING` 或 `TERMINATED` 状态时才会出现。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

### <a name="clusterevent"></a><a id="clusterevent"> </a><a id="clustereventsclusterevent"> </a>ClusterEvent

群集事件信息。

| 字段名称     | 类型                                               | 描述                                                                                                                         |
|----------------|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| cluster_id     | `STRING`                                           | 该群集的规范标识符。 此字段为必需字段。                                                                       |
| timestamp      | `INT64`                                            | 事件发生时的时间戳，存储为自 unix epoch 以来的毫秒数。 由 Timeline 服务分配。 |
| type           | [ClusterEventType](#clustereventsclustereventtype) | 事件类型。 此字段为必需字段。                                                                                             |
| 详细信息        | [EventDetails](#clustereventseventdetails)         | 事件详细信息。 此字段为必需字段。                                                                                          |

### <a name="clustereventtype"></a><a id="clustereventsclustereventtype"> </a><a id="clustereventtype"> </a>ClusterEventType

群集事件的类型。

| 事件类型                      | 说明                                                                                                                                                     |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CREATING                        | 指示正在创建群集。                                                                                                                    |
| DID_NOT_EXPAND_DISK             | 指示磁盘空间不足，但添加磁盘会超出最大容量。                                                                     |
| EXPANDED_DISK                   | 指示磁盘空间不足，并且磁盘已扩展。                                                                                             |
| FAILED_TO_EXPAND_DISK           | 指示磁盘空间不足，并且无法扩展磁盘。                                                                                    |
| INIT_SCRIPTS_STARTING           | 指示群集范围的初始化脚本已启动。                                                                                                      |
| INIT_SCRIPTS_FINISHED           | 指示群集范围的初始化脚本已完成。                                                                                                     |
| STARTING                        | 指示正在启动群集。                                                                                                                    |
| RESTARTING                      | 指示正在重启群集。                                                                                                                    |
| TERMINATING                     | 指示正在终止群集。                                                                                                                 |
| EDITED                          | 指示已编辑了群集。                                                                                                                     |
| RUNNING                         | 指示已完成群集创建。 包括群集中的节点数，以及失败原因（如果无法获取某些节点）。         |
| RESIZING                        | 指示群集的目标大小的变化（变大或变小）。                                                                                      |
| UPSIZE_COMPLETED                | 指示已完成将节点添加到群集。 包括群集中的节点数，以及失败原因（如果无法获取某些节点）。 |
| NODES_LOST                      | 指示某些节点已从群集中丢失。                                                                                                           |
| DRIVER_HEALTHY                  | 指示驱动程序运行正常且群集已可供使用。                                                                                          |
| DRIVER_UNAVAILABLE              | 指示该驱动程序不可用。                                                                                                                       |
| SPARK_EXCEPTION                 | 指示从驱动程序中引发了 Spark 异常。                                                                                                    |
| DRIVER_NOT_RESPONDING           | 指示驱动程序已启动，但没有响应（可能是由于 GC）。                                                                                        |
| DBFS_DOWN                       | 指示驱动程序已启动，但 DBFS 已关闭。                                                                                                               |
| METASTORE_DOWN                  | 指示驱动程序已启动，但元存储已关闭。                                                                                                      |
| NODE_BLACKLISTED                | 指示 Spark 不允许使用某个节点。                                                                                                                  |
| PINNED                          | 指示群集已固定。                                                                                                                          |
| UNPINNED                        | 指示群集已取消固定。                                                                                                                        |

### <a name="eventdetails"></a><a id="clustereventseventdetails"> </a><a id="eventdetails"> </a>EventDetails

有关群集事件的详细信息。

| 字段名称                | 类型                                                            | 描述                                                                                                                                                                           |
|---------------------------|-----------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| current_num_workers       | `INT32`                                                         | 群集中节点的数量。                                                                                                                                                   |
| target_num_workers        | `INT32`                                                         | 群集中节点的目标数量。                                                                                                                                          |
| previous_attributes       | [ClusterAttributes](#clusterclusterattributes)                  | 编辑群集之前的群集属性。                                                                                                                                   |
| attributes                | [ClusterAttributes](#clusterclusterattributes)                  | * 对于已创建的群集，此项为群集的属性。<br>* 对于已编辑的群集，此项为群集的新属性。                                                                   |
| previous_cluster_size     | [clusterSize](#clusterclustersize)                              | 群集在编辑或重设大小之前的大小。                                                                                                                                     |
| cluster_size              | [clusterSize](#clusterclustersize)                              | 在创建或编辑群集时所设置的群集大小。                                                                                                                        |
| cause                     | [ResizeCause](#clustereventsresizecause)                        | 目标大小发生变化的原因。                                                                                                                                                 |
| reason                    | [TerminationReason](#clusterterminationreasonterminationreason) | 终止原因：<br><br>* 对于 `TERMINATED` 事件，此项为终止的原因。<br>* 对于 `RESIZE_COMPLETE` 事件，此项指示未能获取某些节点的原因。 |
| user                      | `STRING`                                                        | 导致事件发生的用户。 （如果事件由 Azure Databricks 导致，则此项为空。）                                                                                                  |

### <a name="clusterattributes"></a><a id="clusterattributes"> </a><a id="clusterclusterattributes"> </a>ClusterAttributes

在群集创建过程中设置的一组常见属性。 在群集的生存期内无法更改这些属性。

| 字段名称                  | 类型                                                        | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-----------------------------|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cluster_name                | `STRING`                                                    | 用户请求的群集名称。 此名称不必唯一。 如果在创建时未指定此字段，群集名称将为空字符串。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| spark_version               | `STRING`                                                    | 群集的运行时版本，例如“5.0.x-scala2.11”。 可以通过使用[运行时版本](#clusterclusterservicelistsparkversions) API 调用来检索可用的运行时版本的列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| spark_conf                  | [SparkConfPair](#clustersparkconfpair)                      | 一个对象，其中包含一组可选的由用户指定的 Spark 配置键值对。 你也可以分别通过以下属性，将额外 JVM 选项的字符串传入到驱动程序和执行程序：<br>`spark.driver.extraJavaOptions` 和 `spark.executor.extraJavaOptions`。<br><br>示例 Spark 配置：<br>`{"spark.speculation": true, "spark.streaming.ui.retainedBatches": 5}` 或<br>`{"spark.driver.extraJavaOptions": "-verbose:gc -XX:+PrintGCDetails"}`                                                                                                                                                                                                                                                          |
| node_type_id                | `STRING`                                                    | 此字段通过单个值对提供给此群集中的每个 Spark 节点的资源进行编码。 例如，可以针对内存密集型或计算密集型的工作负载来预配和优化 Spark 节点。通过使用[列出节点类型](#clusterclusterservicelistnodetypes) API 调用可以检索可用节点类型的列表。                                                                                                                                                                                                                                                                                                                                                                 |
| driver_node_type_id         | `STRING`                                                    | Spark 驱动程序的节点类型。 此字段为可选；如果未设置，驱动程序节点类型将会被设置为与上面定义的 `node_type_id` 相同的值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ssh_public_keys             | 一个由 `STRING` 构成的数组                                        | 将会添加到此群集中各个 Spark 节点的 SSH 公钥内容。 对应的私钥可用于在端口 `2200`上使用用户名 `ubuntu` 登录。 最多可以指定 10 个密钥。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| custom_tags                 | [ClusterTag](#clusterclustertag)                            | 一个对象，其中包含群集资源的一组标记。 Databricks 会使用这些标记以及 default_tags 来标记所有的群集资源。<br><br>**注意**：<br><br>* 旧版节点类型（如计算优化和内存优化）不支持标记<br>* Databricks 最多允许 45 个自定义标记                                                                                                                                                                                                                                                                                                                                                                                                       |
| cluster_log_conf            | [ClusterLogConf](#clusterclusterlogconf)                    | 用于将 Spark 日志发送到长期存储目标的配置。 对于一个群集，只能指定一个目标。 如果提供该配置，日志将会发送到目标，发送的时间间隔为<br>`5 mins`. 驱动程序日志的目标是 `<destination>/<cluster-ID>/driver`，而执行程序日志的目标是 `<destination>/<cluster-ID>/executor`。                                                                                                                                                                                                                                                                                                                                 |
| init_scripts                | 一个由 [InitScriptInfo](#clusterclusterinitscriptinfo) 构成的数组 | 用于存储初始化脚本的配置。 可以指定任意数量的目标。 这些脚本会按照所提供的顺序依次执行。 如果指定了 `cluster_log_conf`，初始化脚本日志将会发送到<br>`<destination>/<cluster-ID>/init_scripts`.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| docker_image                | [DockerImage](#dockerimage)                                 | [自定义容器](../../../clusters/custom-containers.md)的 Docker 映像。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| spark_env_vars              | [SparkEnvPair](#clustersparkenvpair)                        | 一个对象，其中包含一组可选的由用户指定的环境变量键值对。 在启动驱动程序和工作器时，(X,Y) 形式的键值对会按原样导出（即<br>`export X='Y'`）。<br><br>若要额外指定一组 `SPARK_DAEMON_JAVA_OPTS`，建议将其追加到 `$SPARK_DAEMON_JAVA_OPTS`，如以下示例中所示。 这样就确保了还会包含所有默认的 databricks 托管环境变量。<br><br>Spark 环境变量示例：<br>`{"SPARK_WORKER_MEMORY": "28000m", "SPARK_LOCAL_DIRS": "/local_disk0"}` 或<br>`{"SPARK_DAEMON_JAVA_OPTS": "$SPARK_DAEMON_JAVA_OPTS -Dspark.shuffle.service.enabled=true"}` |
| autotermination_minutes     | `INT32`                                                     | 在群集处于不活动状态的时间达到此时间（以分钟为单位）后，自动终止该群集。 如果未设置，将不会自动终止此群集。 如果指定，阈值必须介于 10 到 10000 分钟之间。 你也可以将此值设置为 0，以显式禁用自动终止。                                                                                                                                                                                                                                                                                                                                                                                                                    |
| enable_elastic_disk         | `BOOL`                                                      | 自动缩放本地存储：启用后，此群集在其 Spark 工作器磁盘空间不足时将会动态获取更多磁盘空间。 有关详细信息，请参阅[自动缩放本地存储](../../../clusters/configure.md#autoscaling-local-storage-azure)。                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| instance_pool_id            | `STRING`                                                    | 群集所属的实例池的可选 ID。 有关详细信息，请参阅[池](../../../clusters/instance-pools/index.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| cluster_source              | [ClusterSource](#clusterclustersource)                      | 确定群集是由用户通过 UI 创建的，还是由 Databricks 作业计划程序创建的，还是通过 API 请求创建的。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| policy_id                   | `STRING`                                                    | [群集策略](policies.md) ID。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

### <a name="clustersize"></a><a id="clusterclustersize"> </a><a id="clustersize"> </a>ClusterSize

群集大小规范。

| 字段名称                           | 类型                                      | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|--------------------------------------|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](#clusterautoscale) | 如果是 num_workers，则此项为该群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>在读取群集的属性时，此字段反映的是所需的工作器数，而不是实际的工作器数。 例如，如果将群集的大小从 5 个工作器重设为 10 个工作器，此字段会更新，以反映 10 个工作器的目标大小，而执行程序中列出的工作器将会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。 |

### <a name="listorder"></a><a id="clusterlistorder"> </a><a id="listorder"> </a>ListOrder

基于列表的查询的泛型排序枚举。

| 订单     | 描述                      |
|-----------|----------------------------------|
| DESC      | 降序                 |
| ASC       | 升序                  |

### <a name="resizecause"></a><a id="clustereventsresizecause"> </a><a id="resizecause"> </a>ResizeCause

重设群集大小的原因。

| 原因            | 说明                                                         |
|------------------|---------------------------------------------------------------------|
| AUTOSCALE        | 根据负载自动重设了大小。                                |
| USER_REQUEST     | 用户请求了新的大小。                                          |
| AUTORECOVERY     | 在群集丢失节点后，自动恢复监视器重设了该群集的大小。      |

### <a name="clusterlogconf"></a><a id="clusterclusterlogconf"> </a><a id="clusterlogconf"> </a>ClusterLogConf

群集日志的路径。

| 字段名称            | 类型                                                                                      | 描述                                                                                                                             |
|-----------------------|-------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| dbfs                  | [DbfsStorageInfo](#clusterclusterlogconfdbfsstorageinfo)                                  | 群集日志的 DBFS 位置。 必须提供目标。 例如，应用于对象的<br>`{ "dbfs" : { "destination" : "dbfs:/home/cluster_log" } }` |

### <a name="initscriptinfo"></a><a id="clusterclusterinitscriptinfo"> </a><a id="initscriptinfo"> </a>InitScriptInfo

初始化脚本的路径。 若要了解如何将初始化脚本与 [Databricks 容器服务](../../../clusters/custom-containers.md)配合使用，请参阅[使用初始化脚本](../../../clusters/custom-containers.md#containers-init-script)。

> [!NOTE]
>
> 文件存储类型只适用于使用 [Databricks 容器服务](../../../clusters/custom-containers.md)设置的群集。

| 字段名称            | 类型                                                                                                                                   | 描述                                                                                                                                                                                                                                                                            |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dbfs 或 file          | [DbfsStorageInfo](#clusterclusterinitscriptinfodbfsstorageinfo)<br><br>[FileStorageInfo](#clusterclusterinitscriptinfofilestorageinfo) | 初始化脚本的 DBFS 位置。 必须提供目标。 例如，应用于对象的<br>`{ "dbfs" : { "destination" : "dbfs:/home/init_script" } }`<br><br>初始化脚本的文件位置。 必须提供目标。 例如，应用于对象的<br>`{ "file" : { "destination" : "file:/my/local/file.sh" } }` |

### <a name="clustertag"></a><a id="clusterclustertag"> </a><a id="clustertag"> </a>ClusterTag

群集标记定义。

| 类型           | 描述                                                                                                                                                                                |
|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STRING`       | 标记的键。 该键必须符合以下要求：<br><br>* 长度介于 1 到 512 个字符之间<br>* 未包含 `<>%*&+?\\/` 中的任何字符<br>* 不以 `azure`、`microsoft` 或 `windows` 开头 |
| `STRING`       | 标记的值。 值的长度必须小于或等于 256 个 UTF-8 字符。                                                                                                 |

### <a name="dbfsstorageinfo"></a><a id="clusterclusterinitscriptinfodbfsstorageinfo"> </a><a id="clusterclusterlogconfdbfsstorageinfo"> </a><a id="dbfsstorageinfo"> </a>DbfsStorageInfo

DBFS 存储信息。

| 字段名称      | 类型           | 描述                                      |
|-----------------|----------------|--------------------------------------------------|
| 目标     | `STRING`       | DBFS 目标。 示例： `dbfs:/my/path`       |

### <a name="filestorageinfo"></a><a id="clusterclusterinitscriptinfofilestorageinfo"> </a><a id="filestorageinfo"> </a>FileStorageInfo

文件存储信息。

> [!NOTE]
>
> 此位置类型只适用于使用 [Databricks 容器服务](../../../clusters/custom-containers.md)设置的群集。

| 字段名称      | 类型           | 描述                                         |
|-----------------|----------------|-----------------------------------------------------|
| 目标     | `STRING`       | 文件目标。 示例： `file:/my/file.sh`       |

### <a name="dockerimage"></a>DockerImage

Docker 映像连接信息。

| 字段            | 类型                                | 描述                                               |
|------------------|-------------------------------------|-----------------------------------------------------------|
| url              | 字符串                              | Docker 映像的 URL。                                 |
| basic_auth       | [DockerBasicAuth](#dockerbasicauth) | Docker 存储库的基本身份验证信息。   |

### <a name="dockerbasicauth"></a>DockerBasicAuth

Docker 存储库基本身份验证信息。

| 字段            | 描述                                          |
|------------------|------------------------------------------------------|
| username         | Docker 存储库的用户名。                 |
| password         | Docker 存储库的密码。                  |

### <a name="logsyncstatus"></a><a id="clusterlogsyncstatus"> </a><a id="logsyncstatus"> </a>LogSyncStatus

日志传送状态。

| 字段名称         | 类型           | 描述                                                                                                                      |
|--------------------|----------------|----------------------------------------------------------------------------------------------------------------------------------|
| last_attempted     | `INT64`        | 上次尝试的时间戳。 如果上次尝试失败，last_exception 会包含上次尝试中的异常。             |
| last_exception     | `STRING`       | 上次尝试引发的异常，如果上次尝试时没有异常，则该字段为 NULL（在响应中省略）。 |

### <a name="nodetype"></a><a id="clusternodetype"> </a><a id="nodetype"> </a>NodeType

Spark 节点类型的说明，包括节点的维度以及将会在其上托管该节点的实例类型。

| 字段名称           | 类型                                                          | 描述                                                                                                                                                                                                      |
|----------------------|---------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| node_type_id         | `STRING`                                                      | 此节点类型的唯一标识符。 此字段为必需字段。                                                                                                                                                    |
| memory_mb            | `INT32`                                                       | 可用于此节点类型的内存（以 MB 为单位）。 此字段为必需字段。                                                                                                                                             |
| num_cores            | `FLOAT`                                                       | 可用于此节点类型的 CPU 内核数。 如果计算机实例上的内核数无法被该计算机上 Spark 节点数整除，则此数值可以是小数。 此字段为必需字段。 |
| description          | `STRING`                                                      | 与此节点类型关联的字符串说明。 此字段为必需字段。                                                                                                                                     |
| instance_type_id     | `STRING`                                                      | 运行此节点的硬件类型的标识符。 此字段为必需字段。                                                                                                                           |
| is_deprecated        | `BOOL`                                                        | 该节点类型是否为已弃用。 未弃用的节点类型可提供更高的性能。                                                                                                                        |
| node_info            | [ClusterCloudProviderNodeInfo](#clustercloudprovidernodeinfo) | 云服务提供商报告的节点类型信息。                                                                                                                                                                   |

### <a name="clustercloudprovidernodeinfo"></a>ClusterCloudProviderNodeInfo

有关云服务提供商所提供实例的信息。

| 字段名称                      | 类型                                                              | 说明                                                                                   |
|---------------------------------|-------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| status                          | [ClusterCloudProviderNodeStatus](#clustercloudprovidernodestatus) | 由云服务提供商报告的状态。                                                     |
| available_core_quota            | `INT32`                                                           | 可用的 CPU 内核配额。                                                                     |
| total_core_quota                | `INT32`                                                           | CPU 内核配额总量。                                                                         |

### <a name="clustercloudprovidernodestatus"></a>ClusterCloudProviderNodeStatus

云服务提供商所提供实例的状态。

| 状态                                 | 描述                                          |
|----------------------------------------|------------------------------------------------------|
| NotEnabledOnSubscription               | 不可用于订阅的节点类型。            |
| NotAvailableInRegion                   | 区域中未提供的节点类型。                   |

### <a name="parameterpair"></a><a id="clusterterminationreasonparameterpair"> </a><a id="parameterpair"> </a>ParameterPair

提供有关群集终止原因的更多信息的参数。

| 类型                                                                  | 描述                                           |
|-----------------------------------------------------------------------|-------------------------------------------------------|
| [TerminationParameter](#clusterterminationreasonterminationparameter) | 终止信息的类型。                      |
| `STRING`                                                              | 终止信息。                          |

### <a name="sparkconfpair"></a><a id="clustersparkconfpair"> </a><a id="sparkconfpair"> </a>SparkConfPair

Spark 配置键值对。

| 类型           | 描述                            |
|----------------|----------------------------------------|
| `STRING`       | 配置属性名称。         |
| `STRING`       | 配置属性值。      |

### <a name="sparkenvpair"></a><a id="clustersparkenvpair"> </a><a id="sparkenvpair"> </a>SparkEnvPair

Spark 环境变量键值对。

> [!IMPORTANT]
>
> 在作业群集中指定环境变量时，此数据结构中的字段只接受拉丁字符（ASCII 字符集）。 使用非 ASCII 字符将会返回错误。 例如，中文、日文汉字和表情符号都属于无效的非 ASCII 字符。

| 类型           | 描述                         |
|----------------|-------------------------------------|
| `STRING`       | 环境变量名称。       |
| `STRING`       | 环境变量值。     |

### <a name="sparknode"></a><a id="clustersparkinfosparknode"> </a><a id="sparknode"> </a>SparkNode

Spark 驱动程序或执行程序配置。

| 字段名称              | 类型                                                       | 描述                                                                                                                              |
|-------------------------|------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| private_ip              | `STRING`                                                   | Spark 节点的专用 IP 地址（通常为 10.x.x.x 地址）。 此地址不同于主机实例的专用 IP 地址。 |
| public_dns              | `STRING`                                                   | 此节点的公共 DNS 地址。 此地址可用于访问驱动程序节点上的 Spark JDBC 服务器。                            |
| node_id                 | `STRING`                                                   | 此节点的全局唯一标识符。                                                                                                |
| instance_id             | `STRING`                                                   | 来自云服务提供商的主机实例的全局唯一标识符。                                                                |
| start_timestamp         | `INT64`                                                    | Spark 节点启动时的时间戳（以毫秒表示）。                                                                          |
| host_private_ip         | `STRING`                                                   | 主机实例的专用 IP 地址。                                                                                             |

### <a name="sparkversion"></a><a id="clustersparkversion"> </a><a id="sparkversion"> </a>SparkVersion

群集的 Databricks Runtime 版本。

| 字段名称     | 类型           | 描述                                                                                                                                                                                                                                                                                                                              |
|----------------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| key            | `STRING`       | [Databricks Runtime 版本](index.md#programmatic-version)键，例如 `6.0.x-scala2.11`。 在创建新群集时应以 `spark_version` 形式提供该值。 由于“通配符”版本（即，`6.0.x-scala2.11` 是一个“通配符”版本）存在小 bug 修复，确切的运行时版本可能会随时间的推移而更改。 |
| name           | `STRING`       | 运行时版本的描述性名称，例如“Databricks Runtime 6.0”。                                                                                                                                                                                                                                                        |

### <a name="terminationreason"></a><a id="clusterterminationreasonterminationreason"> </a><a id="terminationreason"> </a>TerminationReason

终止群集的原因。

| 字段名称     | 类型                                                        | 说明                                                                                             |
|----------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| code           | [TerminationCode](#clusterterminationreasonterminationcode) | 指示群集终止原因的状态代码。                                                    |
| type           | [TerminationType](#clusterterminationreasonterminationtype) | 指示为何终止群集的原因。                                                         |
| parameters     | [ParameterPair](#clusterterminationreasonparameterpair)     | 对象，其中包含一组提供群集终止原因相关信息的参数。      |

### <a name="poolclusterterminationcode"></a><a id="clusterterminationreasonpoolclusterterminationcode"> </a><a id="poolclusterterminationcode"> </a>PoolClusterTerminationCode

状态代码，用于指示群集因池故障而终止的原因。

| 代码                                         | 说明                                                                                                            |
|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| INSTANCE_POOL_MAX_CAPACITY_FAILURE           | 已达到池的最大容量。                                                                                |
| INSTANCE_POOL_NOT_FOUND_FAILURE              | 群集指定的池不再处于活动状态或不存在。                                                |

### <a name="clustersource"></a><a id="clusterclustersource"> </a><a id="clustersource"> </a>ClusterSource

创建该群集的服务。

| 服务          | 说明                                          |
|------------------|------------------------------------------------------|
| UI               | 通过 UI 创建的群集。                      |
| JOB              | Databricks 作业计划程序创建的群集。     |
| API              | 通过 API 调用创建的群集。                 |

### <a name="clusterstate"></a><a id="clusterclusterstate"> </a><a id="clusterstate"> </a>ClusterState

群集的状态。 可允许的状态转换如下：

* `PENDING` -> `RUNNING`
* `PENDING` -> `TERMINATING`
* `RUNNING` -> `RESIZING`
* `RUNNING` -> `RESTARTING`
* `RUNNING` -> `TERMINATING`
* `RESTARTING` -> `RUNNING`
* `RESTARTING` -> `TERMINATING`
* `RESIZING` -> `RUNNING`
* `RESIZING` -> `TERMINATING`
* `TERMINATING` -> `TERMINATED`

| 状态             | 描述                                                                                                                                    |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| `PENDING`         | 指示群集正处于创建过程中。                                                                                   |
| `RUNNING`         | 指示群集已启动并已可供使用。                                                                                |
| `RESTARTING`      | 指示群集正处于重启过程中。                                                                                      |
| `RESIZING`        | 指示群集正处于添加或删除节点的过程中。                                                                        |
| `TERMINATING`     | 指示群集正处于销毁过程中。                                                                                 |
| `TERMINATED`      | 指示已成功销毁群集。                                                                                      |
| `ERROR`           | 不会再使用此状态。 此状态曾用于指示未能创建的群集。<br>现已改用 `TERMINATING` 和 `TERMINATED`。 |
| `UNKNOWN`         | 指示群集处于未知状态。 群集永不应处于此状态。                                                      |

### <a name="terminationcode"></a><a id="clusterterminationreasonterminationcode"> </a><a id="terminationcode"> </a>TerminationCode

指示该群集终止原因的状态代码。

| 代码                                 | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| USER_REQUEST                         | 用户已直接终止该群集。 参数应包括一个 `username` 字段，用于指示终止该群集的特定用户。                                                                                                                                                                                                                                                                                                                                                                                                 |
| JOB_FINISHED                         | 群集由某个作业启动，并在该作业完成时终止。                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| INACTIVITY                           | 群集由于处于空闲状态而被终止。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| CLOUD_PROVIDER_SHUTDOWN              | 云服务提供商已终止托管 Spark 驱动程序的实例。                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| COMMUNICATION_LOST                   | Azure Databricks 已丢失与驱动程序实例上服务的连接。 例如，在云网络基础结构中出现问题时，或者，在实例本身运行不正常时，就可能发生这种情况。                                                                                                                                                                                                                                                                                                                                          |
| CLOUD_PROVIDER_LAUNCH_FAILURE        | Azure Databricks 在请求实例以启动群集时遇到了云服务提供商故障。                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| SPARK_STARTUP_FAILURE                | 该群集未能初始化。 可能的原因包括未能为 Spark 创建环境或者在启动 Spark Master 和工作进程时出现问题。                                                                                                                                                                                                                                                                                                                                                                             |
| INVALID_ARGUMENT                     | 无法启动该群集，因为用户指定了无效的参数。  例如，用户可能为该群集指定了无效的运行时版本。                                                                                                                                                                                                                                                                                                                                                                                        |
| UNEXPECTED_LAUNCH_FAILURE            | 在启动此群集时，Azure Databricks 未能完成关键设置步骤，因而导致终止该群集。                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| INTERNAL_ERROR                       | Azure Databricks 遇到了意外错误，该错误已强制终止正在运行的群集。 有关其他详细信息，请联系 Azure Databricks 支持人员。                                                                                                                                                                                                                                                                                                                                                                                       |
| SPARK_ERROR                          | Spark 驱动程序未能启动。 可能的原因包括库不兼容以及初始化脚本损坏了 Spark 容器。                                                                                                                                                                                                                                                                                                                                                                                              |
| METASTORE_COMPONENT_UNHEALTHY        | 群集未能启动，因为无法访问外部元存储。 请参阅[排查故障](../../../data/metastores/external-hive-metastore.md#troubleshooting)。                                                                                                                                                                                                                                                                                                                                                                 |
| DBFS_COMPONENT_UNHEALTHY             | 群集未能启动，因为无法访问 Databricks 文件系统 (DBFS)。                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| AZURE_RESOURCE_PROVIDER_THROTTLING   | Azure Databricks 已达到 Azure 资源提供程序请求限制。 具体而言，是指对特定资源类型（计算、网络等）的 API 请求速率不能超过该限制。 重试可能有助于解决此问题。 有关详细信息，请参阅 [https://docs.microsoft.com/azure/virtual-machines/troubleshooting/troubleshooting-throttling-errors](/virtual-machines/troubleshooting/troubleshooting-throttling-errors)。                                                                              |
| AZURE_RESOURCE_MANAGER_THROTTLING    | Azure Databricks 已达到 Azure 资源管理器请求限制，这将会阻止 Azure SDK 向 Azure 资源管理器发出任何读取或写入请求。 请求限制应用于各个订阅，每小时一次。 请于一小时后重试，或者，更改为较小的群集大小可能会有助于解决此问题。 有关详细信息，请参阅 [https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-request-limits](/azure-resource-manager/resource-manager-request-limits)。 |
| NETWORK_CONFIGURATION_FAILURE        | 由于网络配置中的错误，已终止该群集。 例如，具有 VNet 注入的工作区的 DNS 设置不正确，从而阻止了对工作器项目的访问。                                                                                                                                                                                                                                                                                                                                                         |
| DRIVER_UNREACHABLE                   | Azure Databricks 无法访问 Spark 驱动程序，因为该驱动程序不可访问。                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| DRIVER_UNRESPONSIVE                  | Azure Databricks 无法访问 Spark 驱动程序，因为该驱动程序无响应。                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| INSTANCE_UNREACHABLE                 | Azure Databricks 无法访问实例，因而无法启动群集。 这可能是暂时性的网络问题。 如果问题持续存在，通常表明网络环境配置有误。                                                                                                                                                                                                                                                                                                                             |
| CONTAINER_LAUNCH_FAILURE             | Azure Databricks 无法在群集的工作器节点上启动容器。 请让管理员检查网络配置。                                                                                                                                                                                                                                                                                                                                                                                                               |
| INSTANCE_POOL_CLUSTER_FAILURE        | 池支持的群集特定的故障。 有关详细信息，请参阅[池](../../../clusters/instance-pools/index.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| REQUEST_REJECTED                     | Azure Databricks 此时无法处理该请求。 请稍后重试，如果问题仍然存在，请联系 Azure Databricks。                                                                                                                                                                                                                                                                                                                                                                                                                  |
| INIT_SCRIPT_FAILURE                  | Azure Databricks 无法在群集的某一节点上加载并运行群集范围的初始化脚本，或者该初始化脚本终止时返回了非零退出代码。 请参阅[初始化脚本日志](../../../clusters/init-scripts.md#init-script-log)。                                                                                                                                                                                                                                                                                                         |
| TRIAL_EXPIRED                        | Azure Databricks 试用订阅已过期。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

### <a name="terminationtype"></a><a id="clusterterminationreasonterminationtype"> </a><a id="terminationtype"> </a>TerminationType

群集的终止原因。

| 类型                                 | 描述                                                                                            |
|--------------------------------------|--------------------------------------------------------------------------------------------------------|
| 成功                              | 终止已成功。                                                                                 |
| CLIENT_ERROR                         | 不可重试。 重新尝试创建群集之前，客户端必须先修复参数。                    |
| SERVICE_FAULT                        | Azure Databricks 服务问题。 客户端可以重试。                                                      |
| CLOUD_FAILURE                        | 云服务提供商基础结构问题。 客户端可以在基础问题得到解决之后重试。          |

### <a name="terminationparameter"></a><a id="clusterterminationreasonterminationparameter"> </a><a id="terminationparameter"> </a>TerminationParameter

提供群集终止原因其他相关信息的键。

| 键                             | 描述                                                                                                                                                                                                                                                    |
|---------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| username                        | 终止群集的用户的用户名。                                                                                                                                                                                                           |
| databricks_error_message        | 可能说明群集终止原因的其他上下文。                                                                                                                                                                                        |
| inactivity_duration_min         | 空闲群集在处于非活动状态的时间已达到此持续时间后关闭。                                                                                                                                                                                          |
| instance_id                     | 承载 Spark 驱动程序的实例的 ID。                                                                                                                                                                                                      |
| azure_error_code                | Azure 提供了错误代码，该代码描述了无法预配群集节点的原因。 有关参考信息，请参阅：[https://docs.microsoft.com/azure/virtual-machines/windows/error-messages](/virtual-machines/windows/error-messages)。 |
| azure_error_message             | Azure 中各种不同故障的上下文，可供用户阅读。 此字段是非结构化的，且其确切格式随时可能发生变更。                                                                                                                                  |
| instance_pool_id                | 群集正在使用的实例池的 ID。                                                                                                                                                                                                              |
| instance_pool_error_code        | 特定于某个池的群集故障的[错误代码](#clusterterminationreasonpoolclusterterminationcode)。                                                                                                                                                 |