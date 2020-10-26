---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 管理群集 - Azure Databricks
description: 了解如何管理 Azure Databricks 群集，包括显示、编辑、启动、终止、删除、控制访问权限以及监视性能和日志。
ms.openlocfilehash: cbbccce24e06ca11f68bd5e3704f8fb4698acce0
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121953"
---
# <a name="manage-clusters"></a>管理群集

本文介绍如何管理 Azure Databricks 群集，包括显示、编辑、启动、终止、删除、控制访问权限以及监视性能和日志。

## <a name="display-clusters"></a>显示分类

若要在工作区中显示群集，请单击群集图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。

“群集”页在以下两个选项卡中显示群集：“通用群集”和“作业群集” 。

> [!div class="mx-imgBorder"]
> ![通用群集](../_static/images/clusters/cluster-list-all-purpose-azure.png)

> [!div class="mx-imgBorder"]
> ![作业群集](../_static/images/clusters/cluster-list-jobs-azure.png)

每个选项卡都包括：

* 群集名称
* [State](../dev-tools/api/latest/clusters.md#clusterclusterstate)
* 节点数
* 驱动程序和工作器节点的类型
* Databricks Runtime 版本
* 群集创建者或作业所有者

除了常见的群集信息，“通用群集”选项卡还显示附加到该群集的笔记本 ![附加的笔记本](../_static/images/clusters/cluster-attached-notebooks.png) 的数量。 列表上方是[固定](#cluster-pin)群集的数量。

通用群集名称左侧的图标指示该群集是否固定、该群集是否提供[高并发](configure.md#high-concurrency)群集以及是否已启用[表访问控制](../security/access-control/table-acls/object-privileges.md)：

* Pinned ![Pinned](../_static/images/clusters/cluster-pinned.png)
* 正在启动 ![正在启动](../_static/images/clusters/cluster-starting.png) ，正在终止 ![正在终止](../_static/images/clusters/cluster-terminating.png)
* 标准群集
  * 运行 ![运行](../_static/images/clusters/cluster-running.png)
  * 终止 ![终止](../_static/images/clusters/cluster-terminated.png)
* 高并发性群集
  * 运行 ![无服务器](../_static/images/clusters/cluster-shared.png)
  * 终止 ![无服务器，已终止](../_static/images/clusters/cluster-terminated-shared.png)
* 拒绝访问
  * 运行 ![已锁定](../_static/images/clusters/cluster-locked.png)
  * 终止 ![已锁定，已终止](../_static/images/clusters/cluster-terminated-locked.png)
* 表 ACL 已启用
  * 运行 ![表 ACL](../_static/images/clusters/cluster-table-acl.png)
  * 终止 ![表 ACL 已终止](../_static/images/clusters/cluster-terminated-table-acl.png)

通用群集最右侧的链接和按钮提供对 Spark UI 和日志以及[终止](#cluster-terminate)、[重启](#cluster-start)、[克隆](#cluster-clone)、[权限](#cluster-permissions)和[删除](#cluster-delete)操作的访问权限。

> [!div class="mx-imgBorder"]
> ![群集操作](../_static/images/clusters/interactive-cluster-actions.png)

作业群集最右侧的链接和按钮提供对“作业运行”页、Spark UI 和日志以及[终止](#cluster-terminate)、[克隆](#cluster-clone)和[权限](#cluster-permissions)操作的访问权限。

> [!div class="mx-imgBorder"]
> ![群集操作](../_static/images/clusters/job-cluster-actions.png)

### <a name="filter-cluster-list"></a><a id="filter-cluster-list"> </a><a id="job-cluster"> </a>筛选群集列表

可以使用右上角的按钮和“筛选器”字段筛选群集列表：

> [!div class="mx-imgBorder"]
> ![筛选群集](../_static/images/clusters/cluster-filters.png)

* 若要仅显示你创建的群集，请单击“我创建的群集”。
* 若要仅显示你可以访问的群集（如果已启用[群集访问控制](../security/access-control/cluster-acl.md)），请单击“我可以访问的群集”。
* 若要按任何字段中显示的字符串进行筛选，请在“筛选器”文本框中键入字符串。

## <a name="pin-a-cluster"></a><a id="cluster-pin"> </a><a id="pin-a-cluster"> </a>固定群集

群集在被终止 30 天后会被永久删除。 若要在群集已[终止](#cluster-terminate)超过 30 天后仍保留通用群集配置，[管理员可固定群集](../dev-tools/api/latest/clusters.md#pin)。 最多可以固定 20 个群集。

可从以下位置固定群集：

* 群集列表

  若要固定或取消固定群集，请单击群集名称左侧的固定图标。

  > [!div class="mx-imgBorder"]
  > ![在群集列表中固定群集](../_static/images/clusters/pin-list.png)

* “群集详细信息”页

  若要固定或取消固定群集，请单击群集名称右侧的固定图标。

  > [!div class="mx-imgBorder"]
  > ![在群集详细信息页中固定群集](../_static/images/clusters/pin-detail.png)

还可以调用[固定](../dev-tools/api/latest/clusters.md#clusterclusterservicepincluster) API 终结点，以编程方式固定群集。

## <a name="view-a-cluster-configuration-as-a-json-file"></a><a id="cluster-json"> </a><a id="view-a-cluster-configuration-as-a-json-file"> </a>以 JSON 文件的形式查看群集配置

有时，将群集配置视为 JSON 会很有帮助。 尤其是当你想要使用[群集 API](../dev-tools/api/latest/clusters.md) 创建相似的群集时，此方法特别有用。 查看现有群集时，只需转到“配置”选项卡，单击此选项卡右上角的“JSON”，复制该 JSON 并将其粘贴到 API 调用中 。 JSON 视图为只读。

> [!div class="mx-imgBorder"]
> ![群集配置 JSON](../_static/images/clusters/cluster-json-azure.png)

## <a name="edit-a-cluster"></a><a id="edit-a-cluster"> </a><a id="editing-clusters"> </a>编辑群集

可以从群集详细信息页中编辑群集配置。

> [!div class="mx-imgBorder"]
> ![群集详细信息](../_static/images/clusters/cluster-edit.png)

还可以调用[编辑](../dev-tools/api/latest/clusters.md#clusterclusterserviceeditcluster) API 终结点，以编程方式编辑群集。

> [!NOTE]
>
> * 附加到群集的笔记本和作业在编辑后将保持附加状态。
> * 在群集上安装的库在编辑后将保持安装状态。
> * 如果要编辑正在运行的群集的任何属性（群集大小和权限除外），则必须重启它。 这可能会影响当前正在使用该群集的用户。
> * 只能编辑正在运行或已终止的群集。 但可以在“群集详细信息”页上更新未处于这些状态的群集的权限。

有关可编辑的群集配置属性的详细信息，请参阅[配置群集](configure.md#cluster-configurations)。

## <a name="clone-a-cluster"></a><a id="clone-a-cluster"> </a><a id="cluster-clone"> </a>克隆群集

可以通过克隆现有群集来创建新群集。

* 群集列表

  > [!div class="mx-imgBorder"]
  > ![在群集列表中克隆群集](../_static/images/clusters/clone-list.png)

* “群集详细信息”页

  > [!div class="mx-imgBorder"]
  > ![在“群集详细信息”页中克隆群集](../_static/images/clusters/clone-details.png)

群集创建窗体将打开，其中预填充了群集配置。 以下来自现有群集的属性不包括在克隆中：

* 群集权限
* 已安装的库
* 附加的笔记本

## <a name="control-access-to-clusters"></a><a id="cluster-permissions"> </a><a id="control-access-to-clusters"> </a>控制对群集的访问

群集访问控制允许管理员和委派的用户向其他用户提供细化的群集访问权限。 一般来说，有两种类型的群集访问控制：

1. 群集创建权限：管理员可以选择允许哪些用户创建群集。

   > [!div class="mx-imgBorder"]
   > ![群集创建权限](../_static/images/clusters/acl-allow-user.png)

2. 群集级别权限：具有某群集的“可管理”权限的用户可配置其他用户是否能够通过单击群集操作中的 ![权限图标](../_static/images/access-control/permissions-icon.png) 图标来附加到、重启、管理该群集并调整其大小。

   > [!div class="mx-imgBorder"]
   > ![群集权限](../_static/images/clusters/acl-list.png)

若要了解如何配置群集访问控制和群集级别权限，请参阅[群集访问控制](../security/access-control/cluster-acl.md)。

## <a name="start-a-cluster"></a><a id="cluster-start"> </a><a id="start-a-cluster"> </a>启动群集

除了创建新群集，还可以启动先前[已终止的](#cluster-terminate)群集。 这使你可以使用先前已终止的群集的原始配置对其进行重新创建。

可从以下位置启动群集：

* 群集列表：

  > [!div class="mx-imgBorder"]
  > ![从群集列表启动群集](../_static/images/clusters/start-list.png)

* “群集详细信息”页：

  > [!div class="mx-imgBorder"]
  > ![从“群集详细信息”页启动群集](../_static/images/clusters/start-details.png)

* 笔记本 ![笔记本附加](../_static/images/notebooks/cluster-icon.png) “群集附加”下拉菜单：

  > [!div class="mx-imgBorder"]
  > ![从“笔记本附加”下拉菜单启动群集](../_static/images/clusters/start-from-notebook.png)

还可以调用[启动](../dev-tools/api/latest/clusters.md#clusterclusterservicestartcluster) API 终结点，以编程方式启动群集。

Azure Databricks 标识具有唯一[群集 ID](../dev-tools/api/latest/clusters.md#clusterclusterinfo) 的群集。 启动已终止的群集后，Databricks 将重新创建具有相同 ID 的群集，自动安装所有库，然后重新附加笔记本。

> [!NOTE]
>
> 如果使用的是[试用版工作区](/azure-databricks/quickstart-create-databricks-workspace-portal)并且该试用版已过期，则将无法启动群集。

### <a name="cluster-autostart-for-jobs"></a><a id="autostart-clusters"> </a><a id="cluster-autostart-for-jobs"> </a>针对作业的群集自动启动

当计划运行分配给现有已终止的群集的作业时，或者从 JDBC/ODBC 接口连接到已终止的群集时，该群集将自动重启。  请参阅[创建作业](../jobs.md#job-create)和 [JDBC 连接](../integrations/bi/jdbc-odbc-bi.md#cluster-requirements)。

通过群集自动启动，你可以将群集配置为自动终止，而无需手动干预来为计划的作业重启群集。 此外，还可以通过计划作业在已终止的群集上运行来计划群集初始化。

自动重启群集之前，会检查[群集](../security/access-control/cluster-acl.md)和[作业](../security/access-control/jobs-acl.md)访问控制权限。

> [!NOTE]
>
> 如果群集是在 Azure Databricks 平台版本 2.70 或更早版本中创建的，则不会自动启动：计划在已终止的群集上运行的作业将会失败。

## <a name="terminate-a-cluster"></a><a id="cluster-terminate"> </a><a id="terminate-a-cluster"> </a>终止群集

若要保存群集资源，可以终止群集。 已终止的群集不能运行笔记本或作业，但其配置会进行存储，因此可以在稍后的某个时间点[重用](#cluster-start)（或者在某些类型的作业中[自动启动](#autostart-clusters)）。 可以手动终止群集，也可以将群集配置为在处于不活动状态指定时间后自动终止。 每次终止群集时 Azure Databricks 都会记录信息。

> [!div class="mx-imgBorder"]
> ![终止原因](../_static/images/clusters/termination-reason.png)

> [!NOTE]
>
> 在新的作业群集上运行[作业](../jobs.md)时（建议做法），群集将终止，并且在作业完成后无法重启。 另一方面，如果计划在已终止的现有通用群集上运行作业，则该群集将[自动启动](#autostart-clusters)。

> [!IMPORTANT]
>
> 如果使用的是[试用版高级工作区](/azure-databricks/quickstart-create-databricks-workspace-portal)，所有运行中的群集将在以下情况下终止：
>
> * 将工作区升级到完整的高级版时。
> * 工作区未升级且试用版过期时。

### <a name="manual-termination"></a>手动终止

可从以下位置手动终止群集

* 群集列表

  > [!div class="mx-imgBorder"]
  > ![在群集列表中终止群集](../_static/images/clusters/terminate-list.png)

* “群集详细信息”页

  > [!div class="mx-imgBorder"]
  > ![在群集详细信息页中终止群集](../_static/images/clusters/terminate-details.png)

### <a name="automatic-termination"></a>自动终止

还可以为群集设置自动终止。 在群集创建过程中，可以指定希望群集在处于不活动状态几分钟后终止。 如果当前时间与群集上运行的最后一个命令之间的差值大于处于不活动状态指定时间，则 Azure Databricks 会自动终止该群集。

当群集上的所有命令（包括 Spark 作业、结构化流式处理和 JDBC 调用）执行完毕时，该群集被视为处于不活动状态。

> [!WARNING]
>
> * 群集不会报告使用 DStreams 时产生的活动。 这意味着自动终止的群集在运行 DStreams 时可能会被终止。 请关闭针对运行 DStreams 的群集的自动终止功能，或考虑使用结构化流式处理。
> * 自动终止功能仅监视 Spark 作业，不监视用户定义的本地进程。 因此，如果所有 Spark 作业都已完成，则即使本地进程正在运行，也可能终止群集。

#### <a name="configure-automatic-termination"></a>配置自动终止

可在群集创建页的“Autopilot 选项”框中的“自动终止”字段中配置自动终止 ：

> [!div class="mx-imgBorder"]
> ![自动终止](../_static/images/clusters/autopilot-azure.png)

> [!IMPORTANT]
>
> 自动终止设置的默认值取决于你选择创建[标准群集还是高并发](configure.md#cluster-mode)群集：
>
> * 标准群集配置为在 120 分钟后自动终止。
> * 高并发群集配置为不会自动终止。

可通过清除“自动终止”复选框或通过将处于不活动状态的时间指定为 `0` 来选择退出自动终止。

> [!NOTE]
>
> 自动终止在最新的 Spark 版本中最受支持。 较早的 Spark 版本具有已知的限制，这可能会导致群集活动的报告不准确。 例如，运行 JDBC、R 或流式处理命令的群集可能会报告过时的活动时间，导致群集提前终止。 请升级到最新的 Spark 版本，以从 bug 修补程序和改进的自动终止功能中受益。

### <a name="unexpected-termination"></a>意外终止

有时，群集会意外终止，而不是手动终止或配置的自动终止。 有关终止原因和修正步骤的列表，请参阅[知识库](/databricks/kb/clusters/termination-reasons)。

## <a name="delete-a-cluster"></a><a id="cluster-delete"> </a><a id="delete-a-cluster"> </a>删除群集

删除群集会终止群集并删除其配置。

> [!WARNING]
>
> 不能撤消此操作。

无法删除[固定](#cluster-pin)群集。 若要删除固定群集，必须先由管理员取消固定该群集。

若要删除群集，请在[“作业群集”或“通用群集”选项卡](#display-clusters)上，单击群集操作中的 ![删除图标](../_static/images/clusters/delete-icon.png) 图标。

> [!div class="mx-imgBorder"]
> ![删除群集](../_static/images/clusters/delete-list.png)

还可以调用[永久删除](../dev-tools/api/latest/clusters.md#clusterclusterservicepermanentdeletecluster) API 终结点，以编程方式删除群集。

## <a name="view-cluster-information-in-the-apache-spark-ui"></a><a id="clusters-sparkui"> </a><a id="view-cluster-information-in-the-apache-spark-ui"> </a>在 Apache Spark UI 中查看群集信息

有关 Spark 作业的详细信息显示在 Spark UI 中，可从以下位置进行访问：

* 群集列表：单击群集行上的 Spark UI 链接。
* 群集详细信息页：单击“Spark UI”选项卡。

Spark UI 显示活动群集和已终止群集的群集历史记录。

> [!div class="mx-imgBorder"]
> ![Spark UI](../_static/images/clusters/spark-ui-azure.png)

> [!NOTE]
>
> 如果重启已终止的群集，Spark UI 将显示已重启的群集的信息，而不会显示已终止的群集的历史信息。

## <a name="view-cluster-logs"></a><a id="cluster-logs"> </a><a id="view-cluster-logs"> </a>查看群集日志

Azure Databricks 提供以下三种与群集相关的活动日志记录：

* [群集事件日志](#event-log)，可捕获群集生命周期事件，如创建、终止、配置编辑等。
* [Apache Spark 驱动程序和工作器日志](#driver-logs)，可用于调试。
* [群集 init 脚本日志](init-scripts.md#init-script-log)，这对于调试 init 脚本非常有用。

本部分讨论群集事件日志、驱动程序和工作器日志。 有关 init 脚本日志的详细信息，请参阅 [Init 脚本日志](init-scripts.md#init-script-log)。

### <a name="cluster-event-logs"></a><a id="cluster-event-logs"> </a><a id="event-log"> </a>群集事件日志

群集事件日志显示由用户操作手动触发或 Azure Databricks 自动触发的重要群集生命周期事件。 此类事件影响群集的整个操作以及群集中运行的作业。

有关受支持的事件类型，请参阅 REST API [ClusterEventType](../dev-tools/api/latest/clusters.md#clustereventsclustereventtype) 数据结构。

事件的存储时间为 60 天，相当于 Azure Databricks 中的其他数据保留时间。

#### <a name="view-a-cluster-event-log"></a>查看群集事件日志

1. 单击“群集”图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击群集名称。
3. 单击“事件日志”选项卡。

   > [!div class="mx-imgBorder"]
   > ![事件日志](../_static/images/clusters/cluster-event-log.png)

若要筛选事件，请单击“按事件类型筛选…”字段中的 ![下拉菜单](../_static/images/menu-dropdown.png)， 然后选中一个或多个事件类型复选框。

使用“全选”可通过排除特定事件类型来简化筛选。

> [!div class="mx-imgBorder"]
> ![筛选事件日志](../_static/images/clusters/cluster-event-log-filter.gif)

#### <a name="view-event-details"></a>查看事件详细信息

有关事件的详细信息，请在日志中单击其所在行，然后单击“JSON”选项卡以查看详细信息。

> [!div class="mx-imgBorder"]
> ![事件详细信息](../_static/images/clusters/cluster-event-details.png)

### <a name="cluster-driver-and-worker-logs"></a><a id="cluster-driver-and-worker-logs"> </a><a id="driver-logs"> </a>群集驱动程序和工作器日志

笔记本、作业和库中的直接打印和日志语句会转到 Spark 驱动程序日志。 这些日志包含三个输出：

* 标准输出
* 标准错误
* Log4j 日志

若要从 UI 访问这些驱动程序日志文件，请转到群集详细信息页上的“驱动程序日志”选项卡。

> [!div class="mx-imgBorder"]
> ![驱动程序日志](../_static/images/clusters/driver-logs.png)

日志文件会定期更新。 较早的日志文件显示在页面顶部，其中还列出了时间戳信息。 可以下载任何日志以便进行故障排除。

若要查看 Spark 工作器日志，可以使用 Spark UI。 还可以为群集[配置日志传递位置](configure.md#cluster-log-delivery)。 工作器和群集日志均传递到指定位置。

## <a name="monitor-performance"></a><a id="cluster-performance"> </a><a id="monitor-performance"> </a>监视性能

为了帮助你监视 Azure Databricks 群集的性能，Azure Databricks 提供了从群集详细信息页访问 [Ganglia](http://ganglia.sourceforge.net/) 指标的权限。

此外，你还可以配置 Azure Databricks 群集，以将指标发送到 Azure Monitor（Azure 的监视平台）中的 Log Analytics 工作区。

还可以在群集节点上安装 [Datadog](https://www.datadoghq.com/) 代理，以将 Datadog 指标发送到 Datadog 帐户。

### <a name="ganglia-metrics"></a>Ganglia 指标

若要访问 Ganglia UI，请导航到群集详细信息页上的“指标”选项卡。 Ganglia UI 中提供 CPU 指标，可用于所有 Databricks 运行时。 GPU 指标适用于已启用 GPU 的群集。

> [!div class="mx-imgBorder"]
> ![Ganglia 指标](../_static/images/clusters/metrics-tab.png)

若要查看实时指标，请单击“Ganglia UI”链接。

若要查看历史指标，请单击快照文件。 快照包含所选时间前一小时的聚合指标。

#### <a name="configure-metrics-collection"></a>配置指标集合

默认情况下，Azure Databricks 每 15 分钟收集一次 Ganglia 指标。 若要配置集合时间，请使用 [init 脚本](init-scripts.md)或在[群集创建](../dev-tools/api/latest/clusters.md#clusterclusterservicecreatecluster) API 的 `spark_env_vars` 字段中设置 `DATABRICKS_GANGLIA_SNAPSHOT_PERIOD_MINUTES` 环境变量。

### <a name="azure-monitor"></a><a id="azure-monitor"> </a><a id="azure_monitor"> </a><a id="datadog-metrics"> </a>Azure Monitor

可以配置 Azure Databricks 群集，以将指标发送到 Azure Monitor（Azure 的监视平台）中的 Log Analytics 工作区。 有关完整说明，请参阅[监视 Azure Databricks](https://docs.microsoft.com/azure/architecture/databricks-monitoring/)。

> [!NOTE]
>
> 如果你已在自己的虚拟网络中部署了 Azure Databricks 工作区，并且已将网络安全组 (NSG) 配置为拒绝 Azure Databricks 不需要的所有出站流量，则必须为 AzureMonitor 服务标记配置其他出站规则。

### <a name="datadog-metrics"></a>Datadog 指标

> [!div class="mx-imgBorder"]
> ![Datadog 指标](../_static/images/clusters/datadog-metrics.png)

可以在群集节点上安装 [Datadog](https://www.datadoghq.com/) 代理，以将 Datadog 指标发送到 Datadog 帐户。 以下笔记本演示了如何使用[群集范围 init 脚本](init-scripts.md#cluster-scoped-init-script)在群集上安装 Datadog 代理。

若要在所有群集上安装 Datadog 代理，请在测试群集范围 init 脚本后，使用[全局 init 脚本](init-scripts.md#global-init-script)。

#### <a name="install-datadog-agent-init-script-notebook"></a>安装 Datadog 代理 init 脚本笔记本

[获取笔记本](../_static/notebooks/datadog-init-script.html)