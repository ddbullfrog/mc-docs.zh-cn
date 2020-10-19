---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/15/2020
title: 工作区资产 - Azure Databricks
description: 概括性了解可在 Azure Databricks 工作区中操作的资产。
ms.openlocfilehash: a3c217a4c0fc305ac544b052e0320daacf04e5ce
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937733"
---
# <a name="workspace-assets"></a>工作区资产

本文概述性介绍 Azure Databricks 工作区资产。

## <a name="clusters"></a><a id="clusters"> </a><a id="ws-clusters"> </a>群集

Azure Databricks 群集为各种用例（如运行生产 ETL 管道、流分析、临时分析和机器学习）提供了统一的平台。

有关如何管理和使用群集的详细信息，请参阅[群集](../clusters/index.md)。

## <a name="notebooks"></a><a id="notebooks"> </a><a id="ws-notebooks"> </a>笔记本

笔记本是一种基于 web 的文档界面，其中包含一系列可运行单元（命令），可对文件、[表格](../data/tables.md#tables)、[可视化效果](../notebooks/visualizations/index.md)和叙述性文本进行操作。 命令可以按顺序运行，引用一个或多个以前运行的命令的输出。

笔记本是在 Azure Databricks 中运行代码的一种机制。 另一种机制是[作业](../jobs.md)。

有关如何管理和使用笔记本的详细信息，请参阅[笔记本](../notebooks/index.md)。

## <a name="jobs"></a><a id="jobs"> </a><a id="ws-jobs"> </a>作业

作业是在 Azure Databricks 中运行代码的一种机制。 另一种机制是[笔记本](#ws-notebooks)。

有关如何管理和使用作业的详细信息，请参阅[作业](../jobs.md)。

## <a name="libraries"></a><a id="libraries"> </a><a id="ws-libraries"> </a>库

库使你群集上运行的笔记本和作业能够使用第三方或本地生成的代码。

有关如何管理和使用库的详细信息，请参阅[库](../libraries/index.md)。

## <a name="data"></a><a id="data"> </a><a id="ws-data"> </a>数据

可以将数据导入一个装载到 Azure Databricks 工作区中的分布式文件系统，并在 Azure Databricks 笔记本和群集中使用。 还可以使用各种 Apache Spark 数据源来访问数据。

有关如何管理和使用数据的详细信息，请参阅[数据](../data/index.md#data)。

## <a name="models"></a><a id="models"> </a><a id="ws-models"> </a>模型

模型注册表是一个集中式模型存储，可用于管理 MLflow 模型的完整生命周期。 它提供按时间顺序的模型世系、模型版本控制、阶段转换以及模型和模型版本批注和说明。

有关更多详细信息，请参阅[在 MLflow 模型注册表中管理 MLflow 模型的生命周期](../applications/mlflow/model-registry.md)。

## <a name="experiments"></a><a id="experiments"> </a><a id="ws-experiments"> </a>试验

MLflow 试验是组织的基本构成单位和适用于 MLflow 机器学习模型训练运行的访问控制；所有 MLflow 运行都属于试验。 每个试验都允许可视化、搜索和比较运行，以及下载运行项目或元数据以便在其他工具中进行分析。

有关如何管理和使用试验的详细信息，请参阅[试验](../applications/mlflow/tracking.md#mlflow-experiments)。

## <a name="models"></a>模型

模型指的是 MLflow 已注册的模型，你可使用它通过阶段转换和版本控制在生产中管理 MLflow 模型。 已注册的模型具有唯一的名称、版本、模型世系和其他元数据。

有关如何管理和使用模型的详细信息，请参阅[在 MLflow 模型注册表中管理 MLflow 模型的生命周期](../applications/mlflow/model-registry.md)。