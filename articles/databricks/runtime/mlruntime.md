---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: 用于机器学习的 Databricks Runtime - Azure Databricks
description: 了解 Databricks Runtime ML，它是基于 Databricks Runtime 的运行时，为机器学习和数据科学提供随时可用的环境。
toc-description: Databricks Runtime ML is a variant of Databricks Runtime that adds multiple popular machine learning libraries, including TensorFlow, Keras, PyTorch, and XGBoost.
ms.openlocfilehash: 5e41c02e332420168946422f129979cb8bfb2be5
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937712"
---
# <a name="databricks-runtime-for-machine-learning"></a><a id="databricks-runtime-for-machine-learning"> </a><a id="mlruntime"> </a>用于机器学习的 Databricks Runtime

用于机器学习的 Databricks Runtime (Databricks Runtime ML) 是一个针对机器学习而优化的现成环境。 Databricks Runtime ML 群集包括最常见的机器学习库，例如 TensorFlow、PyTorch、Keras 和 XGBoost，还包括分布式训练所需的库，如 Horovod。 使用 Databricks Runtime ML 可以加快群集创建速度，并确保已安装的库版本兼容。

有关使用 Azure Databricks 进行机器学习和深度学习的完整信息，请参阅[机器学习和深度学习](../applications/machine-learning/index.md)。

有关每个 Databricks Runtime ML 版本的内容的信息，请参阅[发行说明](../release-notes/runtime/releases.md)。

Databricks Runtime ML 基于 Databricks Runtime 构建。 例如，Databricks Runtime 7.3 ML 基于 Databricks Runtime 7.3 构建。
Databricks Runtime [发行说明](../release-notes/runtime/releases.md)中列出了基本 Databricks Runtime 中包含的库。

## <a name="introduction-to-databricks-runtime-for-machine-learning"></a><a id="introduction-to-databricks-runtime-for-machine-learning"> </a><a id="mlversions"> </a>用于机器学习的 Databricks Runtime 的简介

本教程为 Databricks Runtime ML 的新用户设计。 完成此过程大约需要 10 分钟，并显示加载表格数据、训练模型、分布式超参数优化和模型推理的完整端到端示例。 示例还演示了如何使用 MLflow API 和 MLflow 模型注册表。

### <a name="databricks-tutorial-notebook"></a>Databricks 教程笔记本

[获取笔记本](../_static/notebooks/mlflow/mlflow-end-to-end-example-azure.html)

## <a name="libraries-included-in-databricks-runtime-ml"></a><a id="libraries-included-in-databricks-runtime-ml"> </a><a id="mllibraries"> </a>Databricks Runtime ML 中已包含库

> [!NOTE]
>
> [库实用程序](../dev-tools/databricks-utils.md#dbutils-library)在 Databricks Runtime ML 中不可用。

Databricks Runtime ML 包含各种常见的 ML 库。 该库使用每个发行版进行更新，以包括新功能和修复。

Azure Databricks 已将受支持的库的子集指定为顶层库。 对于这些库，Azure Databricks 提供了更快的更新节奏，并使用每个运行时版本更新到最新的包版本（禁止依赖项冲突）。 Azure Databricks 还为顶层库提供高级支持、测试以及嵌入式优化。

有关顶层库和其他提供的库的完整列表，请参阅以下有关每个可用运行时的文章：

* [Databricks Runtime 7.3 ML](../release-notes/runtime/7.3ml.md)
* [Databricks Runtime 7.2 ML](../release-notes/runtime/7.2ml.md)
* [Databricks Runtime 7.1 ML](../release-notes/runtime/7.1ml.md)
* [Databricks Runtime 7.0 ML](../release-notes/runtime/7.0ml.md)
* [Databricks Runtime 6.6 ML](../release-notes/runtime/6.6ml.md)
* [Databricks Runtime 6.5 ML](../release-notes/runtime/6.5ml.md)
* [Databricks Runtime 6.4 ML](../release-notes/runtime/6.4ml.md)
* [Databricks Runtime 5.5 LTS ML](../release-notes/runtime/5.5ml.md)

## <a name="how-to-use-databricks-runtime-ml"></a>如何使用 Databricks Runtime ML

除了预安装的库之外，Databricks Runtime ML 与群集配置中的 Databricks Runtime 和管理 Python 包方式有所不同。

### <a name="create-a-cluster-using-databricks-runtime-ml"></a>使用 Databricks Runtime ML 创建群集

[创建群集](../clusters/create.md#cluster-create)时，请从“Databricks 运行时版本”下拉列表中选择 Databricks Runtime ML 版本。 CPU 和启用 GPU 的 ML 运行时均可用。

> [!div class="mx-imgBorder"]
> ![选择 Databricks Runtime ML](../_static/images/clusters/mlruntime-dbr-dropdown.png)

如果选择已启用 GPU 的 ML 运行时，系统将提示你选择兼容的驱动程序类型和辅助角色类型 。 下拉列表中不兼容的实例类型将灰显。 “GPU 加速”标签下列出了已启用 GPU 的实例类型。

> [!WARNING]
>
> 工作区中[自动安装到所有群集](../libraries/cluster-libraries.md#install-libraries)的库可能与 Databricks Runtime ML 中包含的库冲突。 在使用 Databricks Runtime ML 创建群集之前，为了避免库冲突，请清除“在所有群集上自动安装”复选框。

### <a name="manage-python-packages"></a><a id="conda-pkg"> </a><a id="manage-python-packages"> </a>管理 Python 包

在 Databricks Runtime ML 中，[Conda](https://conda.io/docs/) 包管理器用于安装 Python 包。 所有 Python 包都安装在单个环境中：`/databricks/python2` 在使用 Python 2 的群集上，`/databricks/python3` 在使用 Python 3 群集上。 不支持切换（或激活）Conda 环境。

有关管理 Python 库的信息，请参阅[库](../libraries/index.md)。

## <a name="automl-support"></a>AutoML 支持

Databricks Runtime ML 包括用于自动执行模型开发过程的工具，并帮助你有效地查找性能最佳的模型。

* [托管的 MLFlow](../applications/mlflow/index.md) 管理端到端模型生命周期，包括跟踪试验运行、部署和共享模型以及维护集中式模型注册表。
* [Hyperopt](../applications/machine-learning/automl-hyperparam-tuning/index.md#hyperopt-overview)，扩充了 `SparkTrials` 类，可自动执行并分发 ML 模型参数优化。