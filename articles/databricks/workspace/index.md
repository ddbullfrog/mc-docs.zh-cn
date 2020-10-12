---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: 工作区 - Azure Databricks
description: 了解可以在 Azure Databricks 工作区中访问的资产。
ms.openlocfilehash: 3b424ddaf7c863ff2d71ed0e4cb1c23c207ad11f
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937735"
---
# <a name="workspace"></a>工作区

Azure Databricks 工作区是用于访问所有 Azure Databricks 资产的环境。 工作区将对象（[笔记本](workspace-assets.md#ws-notebooks)、[库](workspace-assets.md#ws-libraries)和[试验](../applications/mlflow/tracking.md#mlflow-experiments)）组织成[文件夹](workspace-objects.md#folders)，并提供对[数据](../data/index.md#data)和计算资源（例如[群集](workspace-assets.md#ws-clusters)和[作业](workspace-assets.md#ws-jobs)）的访问。

> [!div class="mx-imgBorder"]
> ![登陆页面](../_static/images/getting-started/landing-azure.png)

可以使用工作区 UI、[Databricks CLI](../dev-tools/cli/index.md) 和 [Databricks REST API](../dev-tools/api/index.md) 来管理工作区。 Azure Databricks 文档中的大部分文章都侧重于介绍如何使用工作区 UI 执行任务。

若要获取帮助：

1. 单击 ![帮助](../_static/images/help.png) 图标来获取帮助：

   > [!div class="mx-imgBorder"]
   > ![帮助菜单](../_static/images/getting-started/help-menu.png)

2. 选择以下选项之一：
   * **帮助中心**提交帮助票证，并在 Azure Databricks 文档、Azure Databricks 知识库文章、Apache Spark 文档和 Databricks 论坛中进行搜索。
   * 发行说明：查看 Azure Databricks [发行说明](../release-notes/index.md)。
   * 文档：查看 Azure Databricks [文档](/databricks/)。
   * 知识库：查看 Azure Databricks [知识库](/databricks/kb/)。
   * **反馈**：提供 Azure Databricks [产品反馈](https://feedback.azure.com/forums/909463-azure-databricks)。
   * 快捷方式：显示键盘快捷方式[键盘快捷方式](../notebooks/index.md)。

以下文章概述工作区资产、如何使用工作区文件夹和其他对象，以及如何查找工作区和资产的 ID：

* [工作区资产](workspace-assets.md)
* [使用工作区对象](workspace-objects.md)
* [获取工作区、群集、笔记本、模型和作业标识符](workspace-details.md)
* [从旧的区域 URL 迁移到每工作区 URL](migrate-workspace-urls.md)