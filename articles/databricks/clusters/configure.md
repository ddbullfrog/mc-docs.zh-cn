---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 配置群集 - Azure Databricks
description: 了解如何配置 Azure Databricks 群集，包括配置群集模式、运行时、实例类型、大小、池、自动缩放首选项、终止计划、Apache Spark 选项、自定义标记、日志传送等。
ms.openlocfilehash: 446d69a3cd21cdb248ae4e0f7620c03ec5a2116d
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121811"
---
# <a name="configure-clusters"></a><a id="cluster-configurations"> </a><a id="configure-clusters"> </a>配置群集

本文介绍创建和编辑 Azure Databricks 群集时可用的配置选项。 它侧重于说明如何使用 UI 创建和编辑群集。 有关其他方法，请参阅[群集 CLI](../dev-tools/cli/clusters-cli.md) 和[群集 API](../dev-tools/api/latest/clusters.md)。

> [!div class="mx-imgBorder"]
> ![创建群集](../_static/images/clusters/create-dialog-azure.png)

## <a name="cluster-policy"></a>群集策略

[群集策略](../administration-guide/clusters/policies.md)基于一组规则限制配置群集的能力。 策略规则会限制可用于创建群集的属性或属性值。 群集策略的 ACL 可以将策略的使用限制到特定用户和组，因此可以限制你在创建群集时可选择的策略。

若要配置群集策略，请在“策略”下拉列表中选择群集策略。

> [!div class="mx-imgBorder"]
> ![选择群集策略](../_static/images/clusters/policy.png)

> [!NOTE]
>
> 如果尚未[在工作区中创建](../administration-guide/clusters/policies.md#create-a-cluster-policy)任何策略，则不会显示“策略”下拉列表。

如果你有以下权限，则可执行相应的操作：

* 如果有[群集创建权限](../administration-guide/access-control/cluster-acl.md#cluster-create-permission)，则可选择**自由形式**的策略，创建可以充分配置的群集。 **自由形式**的策略不限制任何群集属性或属性值。
* 如果有群集创建权限和群集策略访问权限，则可选择**自由形式**的策略和你有权访问的策略。
* 如果只有群集策略访问权限，则可选择你有权访问的策略。

## <a name="cluster-mode"></a>群集模式

Azure Databricks 支持三种群集模式：标准、高并发和[单节点](single-node.md)。 默认群集模式为“标准”。

> [!NOTE]
>
> 群集配置包括[自动终止](clusters-manage.md#automatic-termination)设置，其默认值依赖于群集模式：
>
> * 标准群集和单节点群集配置为在 120 分钟后自动终止。
> * 高并发群集配置为不会自动终止。

> [!IMPORTANT]
>
> 创建群集后，无法更改群集模式。 如果需要另一群集模式，必须创建新群集。

### <a name="standard-clusters"></a><a id="standard"> </a><a id="standard-clusters"> </a>标准群集

建议在单用户模式下使用标准群集。 标准群集可以运行采用以下任何语言开发的工作负荷：Python、R、Scala 和 SQL。

### <a name="high-concurrency-clusters"></a><a id="high-concurrency"> </a><a id="high-concurrency-clusters"> </a>高并发群集

高并发群集是托管的云资源。 高并发群集的主要优点是，它们提供 Apache Spark 原生的细粒度共享，可以最大限度地提高资源利用率并降低查询延迟。

高并发群集仅支持 SQL、Python 和 R。高并发群集的性能和安全性通过在单独的进程中运行用户代码来实现，这在 Scala 中是不可能的。

此外，只有高并发群集支持[表访问控制](../security/access-control/table-acls/index.md)。

若要创建高并发群集，请在“群集模式”下拉列表中选择“高并发”。

> [!div class="mx-imgBorder"]
> ![高并发群集模式](../_static/images/clusters/high-concurrency.png)

有关如何使用群集 API 创建高并发群集的示例，请参阅[高并发群集示例](../dev-tools/api/latest/examples.md#high-concurrency-example)。

### <a name="single-node-clusters"></a><a id="single-node"> </a><a id="single-node-clusters"> </a>单节点群集

单节点群集没有工作器，在驱动程序节点上运行 Spark 作业。
相比而言，标准模式群集除了需要用于执行 Spark 作业的驱动程序节点外，还需要至少一个 Spark 工作器节点。

若要创建单节点群集，请在“群集模式”下拉列表中选择“单节点”。

> [!div class="mx-imgBorder"]
> ![单节点群集模式](../_static/images/clusters/single-node.png)

若要详细了解如何使用单节点群集，请参阅[单节点群集](single-node.md)。

## <a name="pool"></a>池

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../release-notes/release-types.md)提供。

若要缩短群集启动时间，可以将群集附加到包含空闲实例的预定义[池](instance-pools/index.md)。 附加到池时，群集会从池中分配其驱动程序节点和工作器节点。
如果池中没有足够的空闲资源来满足群集的请求，则池会通过从实例提供程序分配新的实例进行扩展。 终止附加的群集后，它使用的实例会返回到池中，可供其他群集重复使用。

请参阅[使用池](instance-pools/cluster-instance-pool.md#cluster-instance-pool)，详细了解如何在 Azure Databricks 中使用池。

## <a name="databricks-runtime"></a>Databricks Runtime

Databricks 运行时是在[群集](index.md)上运行的核心组件集。 所有 Databricks 运行时都包括 Apache Spark，都添加了可以提高可用性、性能和安全性的组件与更新。

Azure Databricks 提供多种类型的运行时以及这些运行时类型的多个版本，这些版本会在你创建或编辑群集时出现在“Databricks 运行时版本”下拉列表中。

有关详细信息，请参阅 [Databricks 运行时](../runtime/index.md)。

## <a name="python-version"></a><a id="python-3"> </a><a id="python-version"> </a>Python 版本

> [!IMPORTANT]
>
> Python 2 的生命周期已于 2020 年 1 月 1 日结束。 Databricks Runtime 6.0 及更高版本不支持 Python 2。 Databricks Runtime 5.5 及更低版本继续支持 Python 2。

### <a name="python-clusters-running-databricks-runtime-60-and-above"></a>运行 Databricks Runtime 6.0 及更高版本的 Python 群集

[Databricks Runtime 6.0（不受支持）](../release-notes/runtime/6.0.md)及更高版本仅支持 Python 3。 若要了解与 Databricks Runtime 6.0 所引入的 Python 环境相关的重大变更，请参阅发行说明中的 [Python 环境](../release-notes/runtime/6.0.md#dbr6python)。

### <a name="python-clusters-running-databricks-runtime-55-lts"></a>运行 Databricks Runtime 5.5 LTS 的 Python 群集

对于 Databricks Runtime 5.5 LTS，Spark 作业、Python 笔记本单元和库安装都支持 Python 2 和 3。

使用 UI 创建的群集的默认 Python 版本为 Python 3。 在 Databricks Runtime 5.5 LTS 中，使用 REST API 创建的群集的默认版本为 Python 2。

#### <a name="specify-python-version"></a>指定 Python 版本

若要在使用 UI 创建群集时指定 Python 版本，请从“Python 版本”下拉列表中进行选择。

> [!div class="mx-imgBorder"]
> ![群集 Python 版本](../_static/images/clusters/python-select.png)

若要在使用 API 创建群集时指定 Python 版本，请将环境变量 `PYSPARK_PYTHON` 设置为 `/databricks/python/bin/python` 或 `/databricks/python3/bin/python3`。 有关示例，请参阅[创建 Python 3 群集 (Databricks Runtime 5.5 LTS)](../dev-tools/api/latest/examples.md#python-3-example) 中的 REST API 示例。

若要验证 `PYSPARK_PYTHON` 配置是否已生效，请在 Python 笔记本（或 `%python` 单元）中运行以下语句：

```python
import sys
print(sys.version)
```

如果指定了 `/databricks/python3/bin/python3`，其输出应如下所示：

```console
3.5.2 (default, Sep 10 2016, 08:21:44)
[GCC 5.4.0 20160609]
```

> [!IMPORTANT]
>
> 对于 Databricks Runtime 5.5 LTS，当你在笔记本中运行 `%sh python --version` 时，`python` 指的是 Ubuntu 系统 Python 版本，即 Python 2。 使用 `/databricks/python/bin/python` 来指示 Databricks 笔记本和 Spark 使用的 Python 版本：此路径会自动配置为指向正确的 Python 可执行文件。

### <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**是否可以在同一群集上同时使用 Python 2 和 Python 3 笔记本？**

否。 Python 版本是群集范围的设置，不能在每个笔记本上进行配置。

**Python 群集上安装了哪些库？**

若要详细了解已安装的特定库，请参阅 [Databricks 运行时发行说明](../release-notes/runtime/index.md#runtime-release-notes)。

**现有的 PyPI 库是否适用于 Python 3？**

这取决于库的版本是否支持 Python 3 版的 Databricks Runtime 版本。
Databricks Runtime 5.5 LTS 使用 Python 3.5。 Databricks Runtime 6.0 及更高版本和带有 Conda 的 Databricks Runtime 使用 Python 3.7。
特定的旧版 Python 库可能无法与 Python 3.7 前向兼容。
对于这种情况，你需要使用较新版的库。

**现有的 `.egg` 库是否适用于 Python 3？**

这取决于现有的 egg 库是否与 Python 2 和 3 交叉兼容。 如果该库不支持 Python 3，则库附件会出故障，或者会出现运行时错误。

若要全面了解如何将代码移植到 Python 3 以及如何编写与 Python 2 和 3 兼容的代码，请参阅[支持 Python 3](http://python3porting.com/)。

**是否仍可使用初始化脚本安装 Python 库？**

[群集节点初始化脚本](init-scripts.md)的一个常见用例是安装包。
对于 Databricks Runtime 5.5 LTS，请使用 `/databricks/python/bin/pip` 来确保 Python 包安装到 Databricks Python 虚拟环境中，而不是系统 Python 环境中。
对于 Databricks Runtime 6.0 及更高版本和带有 Conda 的 Databricks Runtime，`pip` 命令是指正确的 Python 虚拟环境中的 `pip`。 但是，如果使用初始化脚本来创建 Python 虚拟环境，请始终使用绝对路径来访问 `python` 和 `pip`。

## <a name="cluster-node-type"></a><a id="cluster-node-type"> </a><a id="node-types"> </a>群集节点类型

群集由一个驱动程序节点和多个工作器节点组成。 你可以为驱动程序节点和工作器节点分别选取不同的云提供程序实例类型，虽然默认情况下驱动程序节点使用与工作器节点相同的实例类型。 不同的实例类型系列适用于不同的用例，例如内存密集型工作负荷或计算密集型工作负荷。

> [!NOTE]
>
> 如果安全要求包括[计算隔离](https://docs.microsoft.com/azure/azure-government/azure-secure-isolation-guidance#compute-isolation)，请选择一个 [Standard_F72s_V2](/virtual-machines/fsv2-series) 实例作为工作器类型。 这些实例类型表示使用整个物理主机的隔离虚拟机，并提供为特定工作负荷（例如美国国防部影响级别 5 (IL5) 工作负荷）提供支持所需的隔离级别。

### <a name="driver-node"></a>驱动程序节点

驱动程序保留附加到群集的所有笔记本的状态信息。 驱动程序节点还负责维护 SparkContext，并解释从群集上的某个笔记本或库运行的所有命令。 驱动程序节点还运行与 Spark 执行程序协调的 Apache Spark master。

驱动程序节点类型的默认值与工作器节点类型相同。 如果计划使用 `collect()` 从 Spark 工作器处收集大量数据，并在笔记本中分析这些数据，则可以选择具有更多内存的更大驱动程序节点类型。

> [!TIP]
>
> 由于驱动程序节点保留附加的笔记本的所有状态信息，因此，请务必将未使用的笔记本与驱动程序分离。

### <a name="worker-node"></a>工作器节点

Azure Databricks 工作器运行 Spark 执行程序和正常运行群集所需的其他服务。 通过 Spark 分配工作负荷时，所有分布式处理都在工作器上进行。 Azure Databricks 为每个工作器节点运行一个执行程序；因此，“执行程序”和“工作器”这两个术语可在 Azure Databricks 体系结构的上下文中互换使用。

> [!TIP]
>
> 若要运行 Spark 作业，至少需要一个工作器。 如果群集没有工作器，你可以在驱动程序上运行非 Spark 命令，但 Spark 命令会失败。

### <a name="gpu-instance-types"></a>GPU 实例类型

对于在计算方面富有挑战性并且对性能要求很高的任务（例如与深度学习相关的任务），Azure Databricks 支持通过图形处理单元 (GPU) 进行加速的群集。 此支持为 Beta 版。 有关详细信息，请参阅[支持 GPU 的群集](gpu.md#gpu-clusters)。

## <a name="cluster-size-and-autoscaling"></a><a id="autoscaling"> </a><a id="cluster-size-and-autoscaling"> </a>群集大小和自动缩放

创建 Azure Databricks 群集时，可以为群集提供固定数目的工作器，也可以为群集提供最小数目和最大数目的工作器。

当你提供固定大小的群集时，Azure Databricks 确保你的群集具有指定数量的工作器。 当你为工作器数量提供了范围时，Databricks 会选择运行作业所需的适当数量的工作器。 这称为“自动缩放”。

使用自动缩放，Azure Databricks 可以根据作业特征动态地重新分配工作器。 在计算方面，管道的某些部分可能比其他部分的要求更高，Databricks 会自动在作业的这些阶段添加额外的工作器（并在不再需要它们时将其删除）。

通过自动缩放，可以更轻松地实现高群集利用率，因为无需通过预配群集来匹配工作负载。 这特别适用于其需求随时间变化的工作负荷（例如在一天的过程中浏览数据集），但也可能适用于预配要求未知的、时间较短的一次性工作负荷。 因此，自动缩放有两个优点：

* 与大小恒定且未充分预配的群集相比，工作负荷的运行速度可以更快。
* 与静态大小的群集相比，自动缩放群集可降低总体成本。

自动缩放可以提供这两个优点之一，也可以同时提供这两个优点，具体取决于群集和工作负荷的恒定大小。  当云服务提供商终止实例时，群集大小可能会小于所选工作器的最小数目。 在这种情况下，Azure Databricks 会通过连续重试来重新预配实例，以便保持最小数量的工作器。

> [!NOTE]
>
> 自动缩放不适用于 `spark-submit` 作业。

### <a name="autoscaling-types"></a>自动缩放类型

Azure Databricks 提供两种类型的群集节点自动缩放：标准和优化。 有关优化自动缩放优点的讨论，请参阅关于[优化自动缩放](https://databricks.com/blog/2018/05/02/introducing-databricks-optimized-auto-scaling.html)的博客文章。

自动化（作业）群集始终使用优化自动缩放。 在全用途群集上执行的自动缩放的类型取决于工作区配置。

标准自动缩放由“标准”定价层的工作区中的全用途群集使用。 优化自动缩放由 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中的全用途群集使用。

### <a name="how-autoscaling-behaves"></a>自动缩放的表现方式

自动缩放的表现取决于它是优化自动缩放还是标准自动缩放，以及是应用于全用途群集还是作业群集。

#### <a name="optimized-autoscaling"></a>优化的自动缩放

* 通过 2 个步骤从最小值纵向扩展到最大值。
* 即使群集未处于空闲状态，也可以通过查看 shuffle 文件状态进行纵向缩减。
* 按当前节点数的某个百分比进行纵向缩减。
* 在作业群集上，如果在过去的 40 秒内群集未得到充分利用，则进行纵向缩减。
* 在全用途群集上，如果在过去的 150 秒内群集未得到充分利用，则进行纵向缩减。

#### <a name="standard-autoscaling"></a>标准自动缩放

* 从添加 8 个节点开始。 然后，以指数方式进行纵向扩展，但可能需要执行多个步骤才能达到最大值。你可以通过设置 `spark.databricks.autoscaling.standardFirstStepUp` Spark 配置属性来自定义第一步。
* 仅当群集处于完全空闲状态且在过去 10 分钟内未得到充分利用时，才进行纵向缩减。
* 从 1 个节点开始，以指数方式进行纵向缩减。

### <a name="enable-and-configure-autoscaling"></a>启用并配置自动缩放

若要允许 Azure Databricks 自动重设群集大小，请为群集启用自动缩放，并提供工作器数目的最小值和最大值范围。

1. 启用自动缩放。
   * 全用途群集 - 在“创建群集”页的“Autopilot 选项”框中，选中“启用自动缩放”复选框： 

     > [!div class="mx-imgBorder"]
     > ![启用自动缩放](../_static/images/clusters/autopilot-azure.png)

   * 作业群集 - 在“配置群集”页的“Autopilot 选项”框中，选择“启用自动缩放”复选框： 

   > [!div class="mx-imgBorder"]
   > ![启用自动缩放](../_static/images/clusters/autopilot-job-azure.png)

2. 配置最小和最大工作器数。

   > [!div class="mx-imgBorder"]
   > ![配置最小和最大工作器数](../_static/images/clusters/workers-azure.png)

   > [!IMPORTANT]
   >
   > 如果使用[实例池](instance-pools/cluster-instance-pool.md#cluster-instance-pool)：
   >
   > * 请确保请求的群集大小小于或等于池中[空闲实例的最小数目](instance-pools/configure.md#pool-min)。 如果它大于该数目，则群集启动时间相当于不使用池的群集的启动时间。
   > * 请确保群集最大大小小于或等于池的[最大容量](instance-pools/configure.md#pool-max)。 如果它大于该容量，则无法创建群集。

### <a name="autoscaling-example"></a>自动缩放示例

如果将静态群集重新配置为自动缩放群集，Azure Databricks 会立即在最小值和最大值边界内重设群集的大小，然后开始自动缩放。 例如，下表展示了在将群集重新配置为在 5 到 10 个节点之间进行自动缩放时，具有特定初始大小的群集会发生什么情况。

| 初始大小   | 重新配置后的大小   |
|----------------|------------------------------|
| 6              | 6                            |
| 12             | 10                           |
| 3              | 5                            |

## <a name="autoscaling-local-storage"></a><a id="autoscaling-local-storage"> </a><a id="autoscaling-local-storage-azure"> </a><a id="local-disk-encrypt"> </a>自动缩放本地存储

通常，估算特定作业会占用的磁盘空间量十分困难。 为了让你不必估算在创建时要附加到群集的托管磁盘的 GB 数，Azure Databricks 会自动在所有 Azure Databricks 群集上启用自动缩放本地存储。

自动缩放本地存储时，Azure Databricks 会监视群集的 Spark 工作器上提供的可用磁盘空间量。 如果工作器开始出现磁盘空间严重不足的情况，则 Databricks 会在该工作器的磁盘空间耗尽之前自动将新的托管磁盘附加到该工作器。 附加磁盘时，每个虚拟机的总磁盘空间（包括虚拟机的初始本地存储）存在 5 TB 的限制。

仅当虚拟机返回到 Azure 时，才会拆离附加到虚拟机的托管磁盘。 也就是说，只要虚拟机属于某个正在运行的群集，就永远不会将托管磁盘从该虚拟机中拆离。 若要纵向缩减托管磁盘使用量，Azure Databricks 建议在配置了[群集大小和自动缩放](#autoscaling)或[自动终止](clusters-manage.md#automatic-termination)的群集中使用此功能。

## <a name="spark-configuration"></a><a id="spark-config"> </a><a id="spark-configuration"> </a>Spark 配置

若要微调 Spark 作业，可以在群集配置中提供自定义 [Spark 配置属性](https://spark.apache.org/docs/latest/configuration.html)。

1. 在群集配置页面上，单击“高级选项”切换开关。
2. 单击“Spark”选项卡。

   > [!div class="mx-imgBorder"]
   > ![Spark 配置](../_static/images/clusters/spark-config-azure.png)

使用[群集 API](../dev-tools/api/latest/clusters.md) 配置群集时，请在[创建群集请求](../dev-tools/api/latest/clusters.md#clustercreatecluster)或[编辑群集请求](../dev-tools/api/latest/clusters.md#clustereditcluster)的 `spark_conf` 字段中设置 Spark 属性。

若要为所有群集设置 Spark 属性，请创建一个[全局初始化脚本](init-scripts.md#global-init-script)：

```scala
dbutils.fs.put("dbfs:/databricks/init/set_spark_params.sh","""
  |#!/bin/bash
  |
  |cat << 'EOF' > /databricks/driver/conf/00-custom-spark-driver-defaults.conf
  |[driver] {
  |  "spark.sql.sources.partitionOverwriteMode" = "DYNAMIC"
  |}
  |EOF
  """.stripMargin, true)
```

## <a name="enable-local-disk-encryption"></a>启用本地磁盘加密

> [!NOTE]
>
> 此功能并非适用于所有 Azure Databricks 订阅。 请联系 Microsoft 或 Databricks 客户代表，以申请访问权限。

用于运行群集的某些实例类型可能有本地附加的磁盘。 Azure Databricks 可以在这些本地附加的磁盘上存储 shuffle 数据或临时数据。 为了确保针对所有存储类型加密所有静态数据（包括在群集的本地磁盘上暂时存储的 shuffle 数据），可以启用本地磁盘加密。

> [!IMPORTANT]
>
> 工作负荷的运行速度可能会更慢，因为在本地卷中读取和写入加密的数据会影响性能。

启用本地磁盘加密时，Azure Databricks 会在本地生成一个加密密钥，该密钥特定于每个群集节点，可以用来加密存储在本地磁盘上的所有数据。 此密钥的作用域是每个群集节点的本地，会与群集节点本身一起销毁。 在其生存期内，密钥驻留在内存中进行加密和解密，并以加密形式存储在磁盘上。

若要启用本地磁盘加密，必须使用[群集 API](../dev-tools/api/latest/clusters.md)。 在创建或编辑群集期间，请设置：

```json
{
  "enable_local_disk_encryption": true
}
```

有关如何调用这些 API 的示例，请参阅群集 API 参考中的[创建](../dev-tools/api/latest/clusters.md#clusterclusterservicecreatecluster)和[编辑](../dev-tools/api/latest/clusters.md#clusterclusterserviceeditcluster)。

下面是一个启用本地磁盘加密的群集创建调用示例：

```json
{
  "cluster_name": "my-cluster",
  "spark_version": "6.6.x-scala2.11",
  "node_type_id": "Standard_D3_v2",
  "enable_local_disk_encryption": true,
  "spark_conf": {
    "spark.speculation": true
  },
  "num_workers": 25
}
```

## <a name="environment-variables"></a>环境变量

可以设置你可从群集上运行的脚本访问的环境变量。

1. 在群集配置页面上，单击“高级选项”切换开关。
2. 单击“Spark”选项卡。
3. 在“环境变量”字段中设置环境变量。

   > [!div class="mx-imgBorder"]
   > ![“环境变量”字段](../_static/images/clusters/environment-variables.png)

还可以使用[创建群集请求](../dev-tools/api/latest/clusters.md#clustercreatecluster)或[编辑群集请求](../dev-tools/api/latest/clusters.md#clustereditcluster)群集 API 终结点中的 `spark_env_vars` 字段来设置环境变量。

> [!NOTE]
>
> 在此字段中设置的环境变量在[群集节点初始化脚本](init-scripts.md)中不可用。 初始化脚本仅支持有限的一组预定义[环境变量](init-scripts.md#env-var)。

## <a name="cluster-tags"></a>群集标记

可以使用群集标记轻松地监视组织中各种组所使用的云资源的成本。 你可以在创建群集时将标记指定为键值对，Azure Databricks 会将这些标记应用于 VM 和磁盘卷等云资源。

群集标记将与池标记和工作区（资源组）标记一起传播到这些云资源。 若要详细了解这些标记类型如何协同工作，请参阅[使用群集、池和工作区标记监视使用情况](../administration-guide/account-settings/usage-detail-tags-azure.md)。

为了方便起见，Azure Databricks 对每个群集应用四个默认标记：`Vendor`、`Creator`、`ClusterName` 和 `ClusterId`。

此外，在作业群集上，Azure Databricks 应用两个默认标记：`RunName` 和 `JobId`。

你可以在创建群集时添加自定义标记。 若要配置群集标记，请执行以下步骤：

1. 在群集配置页面上，单击“高级选项”切换开关。
2. 在页面底部，单击“标记”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“标记”选项卡](../_static/images/clusters/tags.png)

3. 为每个自定义标记添加一个键值对。 最多可以添加 43 个自定义标记。

自定义标记显示在 Azure 帐单上，每当你添加、编辑或删除自定义标记时都会进行更新。

## <a name="ssh-access-to-clusters"></a><a id="ssh-access"> </a><a id="ssh-access-to-clusters"> </a>通过 SSH 访问群集

可以通过 [SSH](https://en.wikipedia.org/wiki/Secure_Shell) 以远程方式登录到 Apache Spark 群集，以进行高级故障排除和安装自定义软件的操作。

出于安全原因，Azure Databricks 中的 SSH 端口默认处于关闭状态。 若要允许通过 SSH 对 Spark 群集进行访问，请与 Azure Databricks 支持部门联系。

> [!NOTE]
>
> 仅当你的工作区部署在你自己的 [Azure 虚拟网络](../administration-guide/cloud-configurations/azure/vnet-inject.md)中时，才能启用 SSH。

## <a name="cluster-log-delivery"></a>群集日志传送

创建群集时，可以指定一个位置，用于传送 Spark 驱动程序、工作器和事件日志。
日志每隔五分钟会传送到所选目标一次。 当群集被终止时，Azure Databricks 会确保传送在群集终止之前生成的所有日志。

日志的目标取决于群集 ID。 如果指定的目标为 `dbfs:/cluster-log-delivery`，则 `0630-191345-leap375` 的群集日志会传送到 `dbfs:/cluster-log-delivery/0630-191345-leap375`。

若要配置日志传送位置，请执行以下步骤：

1. 在群集配置页面上，单击“高级选项”切换开关。
2. 在页面底部，单击“日志记录”选项卡。

   > [!div class="mx-imgBorder"]
   > ![群集日志传送](../_static/images/clusters/log-delivery-azure.png)

3. 选择目标类型。
4. 输入群集日志路径。

> [!NOTE]
>
> 此功能在 REST API 中也可用。 请参阅[群集 API](../dev-tools/api/latest/clusters.md) 和[群集日志传送示例](../dev-tools/api/latest/examples.md#cluster-log-example)。

## <a name="init-scripts"></a>初始化脚本

群集节点初始化脚本是一个 shell 脚本，它会在每个群集节点启动期间 Spark 驱动程序或工作器 JVM 启动之前运行。 你可以使用初始化脚本安装未包括在 Databricks 运行时中的包和库、修改 JVM 系统类路径、设置 JVM 所使用的系统属性和环境变量、修改 Spark 配置参数，或者完成其他配置任务。

可以将初始化脚本附加到群集，方法是：展开“高级选项”部分，然后单击“初始化脚本”选项卡。

有关详细说明，请参阅[群集节点初始化脚本](init-scripts.md)。