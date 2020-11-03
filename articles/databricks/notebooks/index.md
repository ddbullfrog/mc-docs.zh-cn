---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/14/2020
title: 笔记本 - Azure Databricks
description: 了解如何在 Azure Databricks 中管理和使用笔记本。
ms.openlocfilehash: 5cc749d159952be5c74ba660ea6392ef008b44b4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106721"
---
# <a name="notebooks"></a>笔记本

笔记本是文档的基于 Web 的接口，其中包含可运行的代码、可视化效果和叙述性文本。

本部分介绍如何管理和使用笔记本。 其中还包含有关创建数据可视化效果、将可视化效果作为仪表板共享、使用小组件参数化笔记本和仪表板、使用笔记本工作流生成复杂管道的文章，以及有关在 Scala 笔记本中定义类的最佳做法。

* [管理笔记本](notebooks-manage.md)
  * [创建笔记本](notebooks-manage.md#create-a-notebook)
  * [打开笔记本](notebooks-manage.md#open-a-notebook)
  * [删除笔记本](notebooks-manage.md#delete-a-notebook)
  * [复制笔记本路径](notebooks-manage.md#copy-notebook-path)
  * [重命名笔记本](notebooks-manage.md#rename-a-notebook)
  * [控制对笔记本的访问](notebooks-manage.md#control-access-to-a-notebook)
  * [笔记本外部格式](notebooks-manage.md#notebook-external-formats)
  * [笔记本和群集](notebooks-manage.md#notebooks-and-clusters)
  * [计划笔记本](notebooks-manage.md#schedule-a-notebook)
  * [分发笔记本](notebooks-manage.md#distribute-notebooks)
* [使用笔记本](notebooks-use.md)
  * [开发笔记本](notebooks-use.md#develop-notebooks)
  * [运行笔记本](notebooks-use.md#run-notebooks)
  * [管理笔记本状态和结果](notebooks-use.md#manage-notebook-state-and-results)
  * [版本控制](notebooks-use.md#version-control)
* [可视化效果](visualizations/index.md)
  * [`display` 函数](visualizations/index.md#display-function)
  * [`displayHTML` 函数](visualizations/index.md#displayhtml-function)
  * [可视化（按语言）](visualizations/index.md#visualizations-by-language)
* [仪表板](dashboards.md)
  * [仪表板笔记本](dashboards.md#dashboards-notebook)
  * [创建计划作业以刷新仪表板](dashboards.md#create-a-scheduled-job-to-refresh-a-dashboard)
  * [查看特定的仪表板版本](dashboards.md#view-a-specific-dashboard-version)
* [小组件](widgets.md)
  * [小组件类型](widgets.md#widget-types)
  * [小组件 API](widgets.md#widget-api)
  * [配置小组件设置](widgets.md#configure-widget-settings)
  * [仪表板中的小组件](widgets.md#widgets-in-dashboards)
  * [将小组件与 %run 配合使用](widgets.md#use-widgets-with-run)
* [笔记本工作流](notebook-workflows.md)
  * [API](notebook-workflows.md#api)
  * [示例](notebook-workflows.md#example)
  * [传递结构化数据](notebook-workflows.md#pass-structured-data)
  * [处理错误](notebook-workflows.md#handle-errors)
  * [同时运行多个笔记本](notebook-workflows.md#run-multiple-notebooks-concurrently)
* [包单元](package-cells.md)
  * [包单元笔记本](package-cells.md#package-cells-notebook)