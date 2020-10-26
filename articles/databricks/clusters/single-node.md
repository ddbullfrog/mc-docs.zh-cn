---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/05/2020
title: 单节点群集 - Azure Databricks
description: 了解 Azure Databricks 单节点群集、何时使用它们、如何创建它们，以及各种限制。
ms.openlocfilehash: 04ae06952cdb75c33f92a7b9e2f57483737b7031
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121858"
---
# <a name="single-node-clusters"></a>单节点群集

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../release-notes/release-types.md)提供。

单节点群集是包含 Spark 驱动程序但不包含 Spark 工作器的群集。 此类群集支持 Spark 作业和所有 Spark 数据源，包括 [Delta Lake](../delta/index.md)。 与之相反，标准群集至少需要一个 Spark 工作器才能运行 Spark 作业。

单节点群集在以下情况下很有用：

* 运行需要 Spark 来加载和保存数据的单节点机器学习工作负荷
* 轻型探索性数据分析 (EDA)

## <a name="create-a-single-node-cluster"></a>创建单节点群集

若要创建单节点群集，请在[配置群集](configure.md#cluster-mode)时在“群集模式”下拉列表中选择“单节点”。

> [!div class="mx-imgBorder"]
> ![单节点群集模式](../_static/images/clusters/single-node.png)

## <a name="single-node-cluster-properties"></a>单节点群集属性

单节点群集具有以下属性：

* 在本地运行 Spark，并使用与群集上的逻辑核心相同数量（驱动程序上的内核数 - 1）的执行程序线程。
* 有 0 个工作器，其中的驱动程序节点同时充当主节点和工作器节点。
* 执行程序 stderr 日志、stdout 日志和 log4j 日志位于驱动程序日志中。
* 不能转换为标准群集， 而只能创建一个模式设置为“标准”的新群集。

## <a name="limitations"></a><a id="limitations"> </a><a id="single-node-limitation"> </a>限制

* 不建议使用单节点群集进行大规模数据处理。 如果超出了单节点群集上的资源，建议使用标准模式群集。
* 建议不要共享单节点群集。 由于所有工作负荷都将在同一节点上运行，因此用户更有可能遇到资源冲突。 Databricks 建议为共享群集使用标准模式。
* 不能通过将最小工作器数量设置为 0 将标准群集转换为单节点群集， 而只能创建一个模式设置为“单节点”的新群集。
* 单节点群集不兼容进程隔离。
* 单节点群集不支持 [Databricks 容器服务](custom-containers.md)。
* 不会在单节点群集上启用 GPU 计划。
* 在单节点群集上，Spark 无法读取具有 UDT 列的 Parquet 文件，并可能返回以下错误消息：

  ```console
  The Spark driver has stopped unexpectedly and is restarting. Your notebook will be automatically reattached.
  ```

  若要解决此问题，请使用以下语句将 Spark 配置 `spark.databricks.io.parquet.nativeReader.enabled` 设置为 `false`：

  ```python
  spark.conf.set("spark.databricks.io.parquet.nativeReader.enabled", False)
  ```

## <a name="single-node-cluster-policy"></a><a id="single-node-cluster-policy"> </a><a id="single-node-policy"> </a>单节点群集策略

[群集策略](../administration-guide/clusters/policies.md)简化了单节点群集的群集配置。

例如，为没有群集创建权限的数据科学团队管理群集时，管理员可能需要授权团队创建总共多达 10 个单节点交互式群集。
这可以使用[实例池](instance-pools/index.md)、[群集策略](../administration-guide/clusters/policies.md)和单节点群集模式来完成：

1. 创建[池](instance-pools/index.md)。 你可以将最大容量设置为 10，启用自动缩放本地存储功能，并选择实例类型和 Databricks Runtime 版本。 记录 URL 中的池 ID。
2. 创建[群集策略](../administration-guide/clusters/policies.md)。 策略中针对实例池 ID 和节点类型 ID 的值应与池属性匹配。 你可以放宽约束以满足你的需求。 请参阅[管理群集策略](../administration-guide/clusters/policies.md)。
3. 向团队成员授予群集策略。 你可以使用[管理用户和组](../administration-guide/users-groups/index.md)来简化用户管理。

   ```json
   {
     "spark_conf.spark.databricks.cluster.profile": {
       "type": "fixed",
       "value": "singleNode",
       "hidden": true
     },
     "instance_pool_id": {
       "type": "fixed",
       "value": "singleNodePoolId1",
       "hidden": true
     },
     "spark_version": {
       "type": "fixed",
       "value": "7.3.x-cpu-ml-scala2.12",
       "hidden": true
     },
     "autotermination_minutes": {
       "type": "fixed",
       "value": 120,
       "hidden": true
     },
     "node_type_id": {
       "type": "fixed",
       "value": "Standard_DS14_v2",
       "hidden": true
     },
     "num_workers": {
       "type": "fixed",
       "value": 0,
       "hidden": true
     }
   }
   ```

## <a name="single-node-job-cluster-policy"></a><a id="single-node-job-cluster-policy"> </a><a id="single-node-policy-job"> </a>单节点作业群集策略

若要设置作业的群集策略，可以定义一个类似的群集策略。
记住将 `cluster_type` 的 “type” 设置为 “fixed”，将 “value” 设置为 “job” 并删除对 `auto_termination_minutes` 的任何引用。

```json
{
  "cluster_type": {
    "type": "fixed",
    "value": "job"
  },
  "spark_conf.spark.databricks.cluster.profile": {
    "type": "forbidden",
    "hidden": true
  },
  "spark_conf.spark.master": {
    "type": "fixed",
    "value": "local[*]"
  },
  "instance_pool_id": {
    "type": "fixed",
    "value": "singleNodePoolId1",
    "hidden": true
  },
  "num_workers": {
    "type": "fixed",
    "value": 0,
    "hidden": true
  },
  "spark_version": {
    "type": "fixed",
    "value": "7.3.x-cpu-ml-scala2.12",
    "hidden": true
  },
  "node_type_id": {
    "type": "fixed",
    "value": "Standard_DS14_v2",
    "hidden": true
  },
  "driver_node_type_id": {
    "type": "fixed",
    "value": "Standard_DS14_v2",
    "hidden": true
  }
}
```