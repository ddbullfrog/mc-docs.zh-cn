---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/15/2020
title: 管理笔记本 - Azure Databricks
description: 了解如何通过使用 UI、CLI 以及通过调用工作区 API 来管理笔记本。
ms.openlocfilehash: 86e748280e58b8a3243541bc007ec4671ed595d3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106672"
---
# <a name="manage-notebooks"></a>管理笔记本

可以通过使用 UI、CLI 以及通过调用工作区 API 来管理笔记本。 本文重点介绍如何使用 UI 执行笔记本任务。 有关其他方法，请参阅 [Databricks CLI](../dev-tools/cli/index.md) 和[工作区 API](../dev-tools/api/latest/workspace.md)。

## <a name="create-a-notebook"></a>创建笔记本

1. 单击边栏中的“工作区”按钮 ![“工作区”图标](../_static/images/workspace/workspace-icon.png) 或“主页”按钮 ![“主页”图标](../_static/images/workspace/home-icon.png)。 执行下列操作之一：
   * 在任何文件夹旁边，单击文本右侧的 ![菜单下拉列表](../_static/images/menu-dropdown.png)，然后选择“创建”>“笔记本”。

     > [!div class="mx-imgBorder"]
     > ![创建笔记本](../_static/images/notebooks/create-notebook.png)

   * 在工作区或用户文件夹中，单击 ![向下的脱字号](../_static/images/down-caret.png)，然后选择“创建”>“笔记本”。
2. 在“创建笔记本”对话框中输入一个名称，然后选择笔记本的默认语言。
3. 如果有正在运行的群集，则会显示“群集”下拉列表。 选择要将笔记本[附加](#attach)到的群集。
4. 单击 **创建** 。

## <a name="open-a-notebook"></a>打开笔记本

在工作区中，单击一个 ![笔记本](../_static/images/access-control/notebook.png). 将鼠标指针悬停在笔记本标题上时，会显示笔记本路径。

## <a name="delete-a-notebook"></a>删除笔记本

请参阅[文件夹](../workspace/workspace-objects.md#folders)和[工作区对象操作](../workspace/workspace-objects.md#objects)，了解如何访问工作区菜单，以及如何删除工作区中的笔记本或其他项。

## <a name="copy-notebook-path"></a>复制笔记本路径

若要在不打开笔记本的情况下复制笔记本文件路径，请右键单击笔记本名称，或者单击笔记本名称右侧的 ![菜单下拉列表](../_static/images/menu-dropdown.png)，然后选择“复制文件路径”。

> [!div class="mx-imgBorder"]
> ![复制笔记本路径](../_static/images/workspace/copy-file-path.png)

## <a name="rename-a-notebook"></a>重命名笔记本

若要更改已打开笔记本的标题，请单击标题并进行内联编辑，或单击“文件”>“重命名”。

## <a name="control-access-to-a-notebook"></a>控制对笔记本的访问

如果 Azure Databricks 帐户有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则可以使用[工作区访问控制](../security/access-control/workspace-acl.md)来控制谁有权访问笔记本。

## <a name="notebook-external-formats"></a><a id="notebook-external-formats"> </a><a id="notebook-formats"> </a>笔记本外部格式

Azure Databricks 支持多种笔记本外部格式：

* 源文件：一个具有 `.scala`、`.py`、`.sql` 或 `.r` 扩展名的文件，其中仅包含源代码语句。
* HTML：一个具有 `.html` 扩展名的 Azure Databricks 笔记本。
* DBC 存档：一个 [Databricks 存档](#databricks-archive)。
* IPython 笔记本：一个具有 `.ipynb` 扩展名的 [Jupyter 笔记本](https://jupyter-notebook.readthedocs.io/en/stable/)。
* RMarkdown：一个具有 `.Rmd` 扩展名的 [R Markdown 文档](https://rmarkdown.rstudio.com/)。

### <a name="in-this-section"></a>本节内容：

* [导入笔记本](#import-a-notebook)
* [导出笔记本](#export-a-notebook)

### <a name="import-a-notebook"></a><a id="import-a-notebook"> </a><a id="import-notebook"> </a>导入笔记本

可以从 URL 或文件导入外部笔记本。

1. 单击边栏中的“工作区”按钮 ![“工作区”图标](../_static/images/workspace/workspace-icon.png) 或“主页”按钮 ![“主页”图标](../_static/images/workspace/home-icon.png)。 执行下列操作之一：
   * 在任意文件夹旁边，单击文本右侧的 ![菜单下拉列表](../_static/images/menu-dropdown.png)，然后选择“导入”。
   * 在工作区或用户文件夹中，单击 ![向下的脱字号](../_static/images/down-caret.png)，然后选择“导入”。

     > [!div class="mx-imgBorder"]
     > ![导入笔记本](../_static/images/notebooks/import-notebook.png)

2. 指定 URL 或浏览到一个包含受支持的外部格式的文件。
3. 单击“导入”。

### <a name="export-a-notebook"></a><a id="export-a-notebook"> </a><a id="export-notebook"> </a>导出笔记本

在笔记本工具栏中，选择“文件”>“导出”和一个[格式](#notebook-formats)。

> [!NOTE]
>
> 如果你将笔记本导出为 HTML、IPython 笔记本或存档 (DBC)，且尚未[清除](notebooks-use.md#clear)结果，则会包含运行该笔记本的结果。

## <a name="notebooks-and-clusters"></a>笔记本和群集

在笔记本中执行任何工作之前，必须先将笔记本附加到群集。 本部分介绍如何在群集中附加和拆离笔记本，以及在执行这些操作时后台会发生什么情况。

### <a name="in-this-section"></a>本节内容：

* [执行上下文](#execution-contexts)
* [将笔记本附加到群集](#attach-a-notebook-to-a-cluster)
* [从群集中拆离笔记本](#detach-a-notebook-from-a-cluster)
* [查看附加到群集的所有笔记本](#view-all-notebooks-attached-to-a-cluster)

### <a name="execution-contexts"></a><a id="execution-context"> </a><a id="execution-contexts"> </a>执行上下文

将笔记本附加到群集时，Azure Databricks 会创建执行上下文。 执行上下文包含以下每种受支持编程语言的 [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) 环境的状态：Python、R、Scala 和 SQL。 当你在笔记本中运行某个单元格时，该命令会调度到相应的语言 REPL 环境并运行。

还可以使用 [REST 1.2 API](../dev-tools/api/1.2/index.md) 来创建执行上下文并发送要在执行上下文中运行的命令。 类似地，该命令会调度到相应的语言 REPL 环境并运行。

群集的执行上下文数量有上限（145 个）。 一旦执行上下文数量达到此阈值，就不能将笔记本附加到群集或创建新的执行上下文。

#### <a name="idle-execution-contexts"></a>空闲的执行上下文

如果上次完成执行后经过的时间超过了已设置的空闲阈值，系统会将执行上下文视为空闲。 上次完成的执行是指笔记本上次执行完命令的时间。 空闲阈值是指在上次完成的执行与任何自动拆离笔记本的尝试之间必须经历的时间。 默认的空闲阈值为 24 小时。

当群集达到最大上下文限制时，Azure Databricks 会根据需要删除（逐出）空闲执行上下文（从最近使用次数最少的开始）。 即使删除了上下文，使用上下文的笔记本仍附加到群集，并显示在群集的笔记本列表中。 流式处理笔记本会被视为正在活跃地运行，其上下文不会被逐出，直到其执行被停止为止。 如果逐出某个空闲上下文，UI 会显示一条消息，指出使用该上下文的笔记本已由于处于空闲状态而被拆离。

> [!div class="mx-imgBorder"]
> ![笔记本上下文被逐出](../_static/images/notebooks/notebook-context-evicted.png)

如果尝试将笔记本附加到已达到执行上下文上限且没有空闲上下文（或禁用了自动逐出）的群集，则 UI 会显示一条消息，指出已达到当前的最大执行上下文阈值，笔记本会保持在已拆离状态。

> [!div class="mx-imgBorder"]
> ![笔记本已拆离](../_static/images/notebooks/notebook-detached.png)

如果为某个进程创建分支，则在为进程创建分支的请求的执行返回后，空闲执行上下文仍会被视为空闲。 使用 Spark 时，建议不要为独立的进程创建分支。

#### <a name="configure-context-auto-eviction"></a><a id="auto-eviction-configure"> </a><a id="configure-context-auto-eviction"> </a>配置上下文自动逐出

可以通过设置 [Spark 属性](../clusters/configure.md#spark-config) `spark.databricks.chauffeur.enableIdleContextTracking` 来配置上下文自动逐出。

* 在 Databricks 5.0 及更高版本中，默认情况下会启用自动逐出。 可以通过设置 `spark.databricks.chauffeur.enableIdleContextTracking false` 来为群集禁用自动逐出。
* 在 Databricks 4.3 中，默认情况下禁用自动逐出。 可以通过设置 `spark.databricks.chauffeur.enableIdleContextTracking true` 来为群集启用自动逐出。

### <a name="attach-a-notebook-to-a-cluster"></a><a id="attach"> </a><a id="attach-a-notebook-to-a-cluster"> </a>将笔记本附加到群集

若要将笔记本附加到群集，请执行以下操作：

1. 在笔记本工具栏中，单击 ![“群集”图标](../_static/images/clusters/clusters-icon.png)“已分离”![群集下拉列表](../_static/images/down-arrow.png)。
2. 从下拉列表中选择一个[群集](../clusters/index.md)。

> [!IMPORTANT]
>
> 附加的笔记本定义了以下 Apache Spark 变量。
>
> | 类                                | 变量名称       |
> |--------------------------------------|---------------------|
> | `SparkContext`                       | `sc`                |
> | `SQLContext`/`HiveContext`           | `sqlContext`        |
> | `SparkSession` (Spark 2.x)           | `spark`             |
>
> 请勿创建 `SparkSession`、`SparkContext` 或 `SQLContext`。 这样做会导致行为不一致。

#### <a name="determine-spark-and-databricks-runtime-version"></a><a id="determine-spark-and-databricks-runtime-version"> </a><a id="variables"> </a>确定 Spark 和 Databricks Runtime 版本

若要确定笔记本附加到的群集的 Spark 版本，请运行：

```python
spark.version
```

若要确定笔记本附加到的群集的 Databricks Runtime 版本，请运行：

##### <a name="scala"></a>Scala

```scala
dbutils.notebook.getContext.tags("sparkVersion")
```

##### <a name="python"></a>Python

```python
spark.conf.get("spark.databricks.clusterUsageTags.sparkVersion")
```

> [!NOTE]
>
> 此 `sparkVersion` 标记以及[群集 API](../dev-tools/api/latest/clusters.md) 和[作业 API](../dev-tools/api/latest/jobs.md) 中的终结点所需的 `spark_version` 属性均指的是 [Databricks Runtime 版本](../dev-tools/api/latest/index.md#programmatic-version)（而不是 Spark 版本）。

### <a name="detach-a-notebook-from-a-cluster"></a><a id="detach"> </a><a id="detach-a-notebook-from-a-cluster"> </a>从群集中拆离笔记本

1. 在笔记本工具栏中，单击 ![“群集”图标](../_static/images/clusters/clusters-icon.png)“已附加 <cluster-name>”![群集下拉列表](../_static/images/down-arrow.png)。
2. 选择“拆离”。

   > [!div class="mx-imgBorder"]
   > ![拆离笔记本](../_static/images/notebooks/cluster-detach.png)

也可使用群集详细信息页上的“笔记本”选项卡将笔记本从群集中拆离。

将笔记本从群集中拆离时，会删除[执行上下文](#execution-context)，并会从笔记本中清除已计算出来的所有变量值。

> [!TIP]
>
> Azure Databricks 建议从群集中拆离未使用的笔记本。 这将释放驱动程序占用的内存空间。

### <a name="view-all-notebooks-attached-to-a-cluster"></a>查看附加到群集的所有笔记本

群集详细信息页上的“笔记本”选项卡会显示附加到群集的所有笔记本。 该选项卡还显示每个已附加的笔记本的状态，以及上次在笔记本中运行命令的时间。

> [!div class="mx-imgBorder"]
> ![群集详细信息 - 附加的笔记本](../_static/images/clusters/notebooks.png)

## <a name="schedule-a-notebook"></a><a id="schedule-a-notebook"> </a><a id="schedule-notebook"> </a>计划笔记本

若要计划一个需定期运行的笔记本作业，请执行以下操作：

1. 在笔记本工具栏中，单击右上方的 ![计划](../_static/images/notebooks/schedule.png) 按钮。
2. 单击“+ 新建”。
3. 选择该计划。
4. 单击“确定”。

## <a name="distribute-notebooks"></a><a id="databricks-archive"> </a><a id="distribute-notebooks"> </a>分发笔记本

为了让你能够轻松地分发 Azure Databricks [笔记本](#notebook-formats)，Azure Databricks 支持了 Databricks 存档（一个包，其中可以包含笔记本的文件夹或单个笔记本）。 Databricks 存档是一个具有额外元数据的 JAR 文件，其扩展名为 `.dbc`。 存档中包含的笔记本采用 Azure Databricks 内部格式。

### <a name="import-an-archive"></a>导入存档

1. 单击文件夹或笔记本右侧的 ![向下的脱字号](../_static/images/down-caret.png) 或 ![菜单下拉菜单](../_static/images/menu-dropdown.png)，然后选择“导入”。
2. 选择“文件”或“URL”。
3. 转到放置区中的 Databricks 存档，或在放置区中放置一个该存档。
4. 单击“导入”。 存档会导入到 Azure Databricks 中。 如果存档包含文件夹，Azure Databricks 会重新创建该文件夹。

### <a name="export-an-archive"></a>导出存档

单击文件夹或笔记本右侧的 ![向下的脱字号](../_static/images/down-caret.png) 或 ![菜单下拉菜单](../_static/images/menu-dropdown.png)，然后选择“导出”>“DBC 存档”。 此时 Azure Databricks 会下载名为 `<[folder|notebook]-name>.dbc` 的文件。