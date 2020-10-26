---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/15/2020
title: 池配置 - Azure Databricks
description: 了解 Azure Databricks 池配置。
ms.openlocfilehash: 06135ab1acec78a09d6ee200bfcb055381f4770d
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121794"
---
# <a name="pool-configurations"></a><a id="instance-pool-configurations"> </a><a id="pool-configurations"> </a>池配置

本文介绍了创建和编辑池时可用的配置选项。

> [!div class="mx-imgBorder"]
> ![配置池](../../_static/images/instance-pools/create-dialog-azure.png)

## <a name="pool-size-and-auto-termination"></a><a id="instance-pool-configuration"> </a><a id="instance-pool-sizing"> </a><a id="pool-size-and-auto-termination"> </a>池大小和自动终止

创建池时，若要控制其大小，你可以设置三个参数：最小空闲实例数、最大容量和空闲实例自动终止。

### <a name="minimum-idle-instances"></a><a id="minimum-idle-instances"> </a><a id="pool-min"> </a>最小空闲实例数

池保持空闲状态的最小实例数。 无论“空闲实例自动终止”中指定的设置如何，这些实例都不会终止。 如果群集使用池中的空闲实例，则 Azure Databricks 会预配更多的实例，以维持此最小值。

> [!div class="mx-imgBorder"]
> ![最小空闲实例数配置](../../_static/images/instance-pools/min-idle.png)

### <a name="maximum-capacity"></a><a id="maximum-capacity"> </a><a id="pool-max"> </a>最大容量

池将预配的最大实例数。 如果设置了此项，则此值约束所有实例（空闲 + 已使用）。
如果使用池的群集在[自动缩放](../configure.md#autoscaling)期间请求比此数目更多的实例，则请求会失败并出现 `INSTANCE_POOL_MAX_CAPACITY_FAILURE` 错误。

> [!div class="mx-imgBorder"]
> ![最大容量配置](../../_static/images/instance-pools/max-capacity.png)

此配置是可选的。 Azure Databricks 建议仅在以下情况下设置值：

* 你有一个不能超过的实例配额。
* 你想要防止一组工作影响另一组工作。 例如，假设你的实例配额为 100，你的团队 A 和 B 需要运行作业。 你可以创建最大配额为 50 的池 A 和最大配额为 50 的池 B，以便两个团队公平地共享配额 100。
* 你需要控制成本。

### <a name="idle-instance-auto-termination"></a>空闲实例自动终止

超出[最小空闲实例数](#pool-min)中设置的值的实例在被池终止之前可以空闲的时间（以分钟为单位）。

> [!div class="mx-imgBorder"]
> ![空闲实例自动终止配置](../../_static/images/instance-pools/autotermination.png)

## <a name="instance-types"></a><a id="instance-pool-types"> </a><a id="instance-types"> </a>实例类型

池由为新群集准备好的空闲实例和正在运行的群集使用的实例组成。 所有这些实例都属于相同的实例提供程序类型，该类型是在创建池时选择的。

无法编辑池的实例类型。 附加到池的群集为驱动程序和工作器节点使用相同的实例类型。 不同的实例类型系列适用于不同的用例，例如内存密集型工作负荷或计算密集型工作负荷。

> [!div class="mx-imgBorder"]
> ![实例类型](../../_static/images/instance-pools/instance-type-azure.png)

Azure Databricks 在停止支持实例类型之前，始终会提供为期一年的弃用通知。

> [!NOTE]
>
> 如果安全要求包括[计算隔离](https://docs.microsoft.com/azure/azure-government/azure-secure-isolation-guidance#compute-isolation)，请选择一个 [Standard_F72s_V2](/virtual-machines/fsv2-series) 实例作为工作器类型。 这些实例类型表示使用整个物理主机的隔离虚拟机，并提供为特定工作负荷（例如美国国防部影响级别 5 (IL5) 工作负荷）提供支持所需的隔离级别。

## <a name="preload-databricks-runtime-version"></a><a id="preload-databricks-runtime-version"> </a><a id="preloaded-spark"> </a>预加载的 Databricks Runtime 版本

可以通过选择要在池中空闲实例上加载的 Databricks Runtime 版本来加快群集启动。  如果用户在创建由池支持的群集时选择了该运行时，则该群集甚至会比未使用预加载 Databricks Runtime 版本的池支持的群集更快地启动。

> [!div class="mx-imgBorder"]
> ![预加载的的运行时版本](../../_static/images/instance-pools/preloaded-spark.png)

## <a name="pool-tags"></a><a id="instance-pool-tags"> </a><a id="pool-tags"> </a>池标记

可以使用池标记轻松地监视组织中各种组所使用的云资源的成本。 你可以在创建池时将标记指定为键值对，Azure Databricks 会将这些标记应用于 VM 和磁盘卷等云资源。

为了方便起见，Azure Databricks 对每个池应用三个默认标记：`Vendor`、`DatabricksInstancePoolId` 和 `DatabricksInstancePoolCreatorId`。 你还可以在创建池时添加自定义标记。 最多可以添加 41 个自定义标记。

### <a name="custom-tag-inheritance"></a>自定义标记继承

池支持的群集从池配置继承默认的和自定义的标记。 若要详细了解池标记和群集标记如何协同工作，请参阅[使用群集、池和工作区标记监视使用情况](../../administration-guide/account-settings/usage-detail-tags-azure.md)。

### <a name="configure-custom-pool-tags"></a>配置自定义池标记

1. 在池配置页面的底部，选择“标记”选项卡。
2. 为自定义标记指定一个键值对。

   > [!div class="mx-imgBorder"]
   > ![标记键值对](../../_static/images/instance-pools/tags.png)

3. 单击“添加” 。

## <a name="autoscaling-local-storage"></a><a id="autoscaling-local-storage"> </a><a id="instance-pools-autoscaling-local-storage-azure"> </a>自动缩放本地存储

通常，估算特定作业会占用的磁盘空间量十分困难。 为了让你不必估算在创建时要附加到池的托管磁盘的 GB 数，Azure Databricks 会自动在所有 Azure Databricks 池上启用自动缩放本地存储。

自动缩放本地存储时，Azure Databricks 会监视池的实例上提供的可用磁盘空间量。 如果某个实例的磁盘空间太少，系统会在该实例的磁盘空间不足之前自动附加新的托管磁盘。 附加磁盘时，每个虚拟机的总磁盘空间（包括虚拟机的初始本地存储）存在 5 TB 的限制。

仅当虚拟机返回到 Azure 时，才会拆离附加到虚拟机的托管磁盘。 也就是说，只要虚拟机属于某个池，就永远不会将托管磁盘从该虚拟机中拆离。