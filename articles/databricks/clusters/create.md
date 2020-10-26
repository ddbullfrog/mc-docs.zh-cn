---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/09/2020
title: 创建群集 - Azure Databricks
description: 了解如何创建 Azure Databricks 群集。
ms.openlocfilehash: 862f63eb37c24e85506978cabe39152a6c7ed07e
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121808"
---
# <a name="create-a-cluster"></a><a id="cluster-create"> </a><a id="create-a-cluster"> </a>创建群集

可以通过两种方式使用 UI 创建群集：

* 创建可由多个用户共享的_通用群集_。 这些群集通常用于运行笔记本。 通用群集会保持活动状态，直到你终止它们为止。
* 创建_作业群集_以运行作业。 创建作业时，会创建作业群集。 作业完成后，此类群集会自动终止。

本文介绍如何使用 UI 创建通用群集。 若要了解如何创建作业群集，请参阅[创建作业](../jobs.md#job-create)。

> [!NOTE]
>
> 你必须有创建群集的权限。 请参阅[配置群集创建权限](../administration-guide/access-control/cluster-acl.md#cluster-create-permission)。

若要使用 UI 创建群集，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“创建群集”按钮。****

   > [!div class="mx-imgBorder"]
   > ![创建群集](../_static/images/clusters/create.png)

   > [!NOTE]
   >
   > 如果使用的是[试用工作区](/azure-databricks/quickstart-create-databricks-workspace-portal)，并且试用期已过，则“创建群集”按钮会被禁用，你将无法创建群集。

3. 命名和配置群集。

   有许多群集配置选项，在[群集配置](configure.md#cluster-configurations)中对其进行了详细说明。

   > [!div class="mx-imgBorder"]
   > ![群集配置](../_static/images/clusters/create-dialog-azure.png)

4. 单击“创建”  按钮。

   群集的“配置”选项卡在群集处于 `pending` 状态时会显示一个旋转的进度指示器。

   > [!div class="mx-imgBorder"]
   > ![群集挂起状态](../_static/images/clusters/pending-spinner.png)

   当群集已启动并准备好使用时，进度旋转图标会变成一个实心的绿色圆圈：

   > [!div class="mx-imgBorder"]
   > ![群集状态](../_static/images/clusters/cluster-ready.png)

   此项指示群集处于 `running` 状态，现在可以附加笔记本并开始运行命令和查询。