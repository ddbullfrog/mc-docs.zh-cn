---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/31/2020
title: 群集库 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用和管理基于群集的库。
ms.openlocfilehash: ffe37f3c7d43a407582b3a7eb0b436cceb38c607
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937829"
---
# <a name="cluster-libraries"></a>群集库

群集库可供群集上运行的所有笔记本使用。 可以使用以前安装的工作区库或使用初始化脚本，直接从公共存储库（例如 PyPI 或 Maven）安装群集库。

## <a name="install-a-library-on-a-cluster"></a><a id="install-a-library-on-a-cluster"> </a><a id="install-libraries"> </a>在群集上安装库

可以通过两种主要方式在群集上安装库：

* 安装已上传到工作区的[工作区库](workspace-libraries.md)。
* 安装仅与特定群集配合使用的库。

此外，如果你的库需要自定义配置，则你可能无法使用上面列出的方法来安装它。 但是，可以使用在创建群集时运行的[初始化脚本](#init-script)来安装该库。

> [!NOTE]
>
> 在群集上安装库时，已连接到该群集的笔记本不会立即看到新库。 必须先[拆离](../notebooks/notebooks-manage.md#detach)笔记本，然后将笔记本[重新附加](../notebooks/notebooks-manage.md#attach)到群集。

### <a name="in-this-section"></a>本节内容：

* [工作区库](#workspace-library)
* [群集安装的库](#cluster-installed-library)
* [初始化脚本](#init-script)

### <a name="workspace-library"></a><a id="install-workspace-libraries"> </a><a id="workspace-library"> </a>工作区库

> [!NOTE]
>
> 从 Databricks Runtime 7.2 开始，Azure Databricks 按照在群集上的安装顺序处理所有工作区库。 在 Databricks Runtime 7.1 及更低版本上，Azure Databricks 按照在群集上的安装顺序处理 Maven 和 CRAN 库。
>
> 如果库之间存在依赖关系，则可能需要注意群集上的安装顺序。

若要安装工作区中已存在的库，可以从群集 UI 或库 UI 开始：

#### <a name="cluster"></a>群集

1. 单击“群集”图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击群集名称。
3. 单击 **“库”** 选项卡。
4. 单击“新安装”。
5. 在“库源”按钮列表中，选择“工作区”。
6. 选择一个工作区库。
7. 单击“安装”  。
8. 若要配置要安装在所有群集上的库，请执行以下操作：
   1. 单击该库。
   1. 选中“在所有群集上自动安装”复选框。
   1. 单击“确认”  。

#### <a name="library"></a>库

1. 转到包含该库的文件夹。
2. 单击库名称。
3. 执行下列操作之一：
   * 若要配置要安装在所有群集上的库，请选中“在所有群集上自动安装”复选框，然后单击“确认”。

     > [!IMPORTANT]
     >
     > 此选项不会在运行 Databricks Runtime 7.0 及更高版本的群集上安装该库。

   * 选中要在其上安装该库的群集旁边的复选框，然后单击“安装”。

该库将安装在此群集上。

### <a name="cluster-installed-library"></a>群集安装的库

可以将库安装在特定群集上，而不必将其用作工作区库。

若要在群集上安装库，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击群集名称。
3. 单击 **“库”** 选项卡。
4. 单击“新安装”。
5. 按照创建[工作区库](workspace-libraries.md)的方法之一进行操作。 单击“创建”后，该库将安装在群集上。

### <a name="init-script"></a>初始化脚本

如果库需要自定义配置，则可能无法使用工作区或群集库界面进行安装。 可以改用[初始化脚本](../clusters/init-scripts.md)安装该库。

下面以某个初始化脚本为例，说明了如何使用 [Conda](https://conda.io/docs/) 包管理器在群集初始化时在用于机器学习的 Databricks Runtime 群集上安装 Python 库。 （Conda 只能在 Databricks Runtime ML 上使用，而不能在基本 Databricks Runtime 上使用）：

```bash
#!/bin/bash
set -ex
/databricks/python/bin/python -V
. /databricks/conda/etc/profile.d/conda.sh
conda activate /databricks/python
conda install -y astropy
```

## <a name="uninstall-a-library-from-a-cluster"></a><a id="uninstall-a-library-from-a-cluster"> </a><a id="uninstall-libraries"> </a>从群集中卸载库

> [!NOTE]
>
> 从群集中卸载库时，仅在重启群集时才会删除该库。 在重启群集之前，已卸载库的状态显示为“卸载等待重启”。

若要卸载库，可以从群集或库开始操作：

### <a name="cluster"></a>群集

1. 单击“群集”图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击群集名称。
3. 单击 **“库”** 选项卡。
4. 选中要从中卸载库的群集旁边的复选框，然后依次单击“卸载”、“确认”。 状态将更改为“卸载等待重启”。

### <a name="library"></a>库

1. 转到包含该库的文件夹。
2. 单击库名称。
3. 选中要从中卸载库的群集旁边的复选框，然后依次单击“卸载”、“确认”。 状态将更改为“卸载等待重启”。
4. 单击群集名称以转到群集详细信息页。

单击“重启”和“确认”以卸载该库。 该库将从群集的“库”选项卡中删除。

## <a name="view-the-libraries-installed-on-a-cluster"></a>查看群集上安装的库

1. 单击“群集”图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击群集名称。
3. 单击“库”选项卡。对于每个库，该选项卡显示名称和版本、类型、[安装状态](../dev-tools/api/latest/libraries.md#managedlibrarieslibraryinstallstatus)以及源文件（如果已上传）。

## <a name="update-a-cluster-installed-library"></a>更新群集安装的库

若要更新群集安装的库，请卸载旧版本的库，然后安装新版本。