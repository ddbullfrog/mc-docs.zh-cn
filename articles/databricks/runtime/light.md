---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: Databricks Light - Azure Databricks
description: 了解 Databricks Light，这是开放源代码 Apache Spark 运行时的 Databricks 包。
toc-description: Databricks Light provides a runtime option for jobs that don’t need the advanced performance, reliability, or autoscaling benefits provided by Databricks Runtime.
ms.openlocfilehash: 1d53c973eebcec68dad76a5aa8c1e311ef175944
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937791"
---
# <a name="databricks-light"></a><a id="databricks-light"> </a><a id="light"> </a>Databricks Light

Databricks Light 是开放源代码 Apache Spark 运行时的 Databricks 包。 它为不需要 Databricks Runtime 所提供的高级性能、可靠性或自动缩放优势的作业提供运行时选项。 特别是 Databricks Light 不支持：

* Delta Lake
* Autopilot 功能，如自动缩放
* 高度并发，通用群集
* 笔记本、仪表板和协作功能
* 用于连接各种数据源和 BI 工具的连接器

Databricks Light 是作业（或“自动化工作负荷”）的运行时环境。 在 Databricks Light 群集上运行作业时，它们会受到较低的作业轻量计算定价的限制。 仅当创建或计划某个 [JAR、Python 或 spark-submit](../dev-tools/api/latest/jobs.md#jobscreatejob) 作业并将某个群集附加到该作业时，才可以选择 Databricks Light；不能使用 Databricks Light 运行笔记本作业或交互式工作负荷。

Databricks Light 可在与其他 Databricks 运行时和定价层上运行的群集相同的工作区中使用。 无需请求单独的工作区即可开始使用。

## <a name="whats-in-databricks-light"></a>Databricks Light 中有什么内容？

Databricks Light 运行时的发布计划遵循 Apache Spark 运行时发布计划。 任何 Databricks Light 版本都基于特定版本的 Apache Spark。 有关详细信息，请参阅以下发行说明：

* [Databricks Light 2.4](../release-notes/runtime/2.4light.md)

## <a name="create-a-cluster-using-databricks-light"></a>使用 Databricks Light 创建群集

创建作业[群集](../jobs.md#job-create)时，请从“Databricks Runtime 版本”下拉列表中选择 Databricks Light 版本。

> [!IMPORTANT]
>
> [公共预览版](../release-notes/release-types.md)提供对[池支持的](../clusters/instance-pools/index.md)作业群集上 Databricks Light 的支持。

> [!div class="mx-imgBorder"]
> ![选择 Databricks Light](../_static/images/jobs/light-runtime.png)