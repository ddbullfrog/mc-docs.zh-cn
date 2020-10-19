---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/29/2020
title: 适用于 Python 开发人员的 Azure Databricks - Azure Databricks
description: 了解如何使用 Python 在 Azure Databricks 中进行开发。
ms.openlocfilehash: cd3c9d5bca4f2de1b84a78b45a84f875802b6871
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937832"
---
# <a name="azure-databricks-for-python-developers"></a>适用于 Python 开发人员的 Azure Databricks

本部分指导你使用 Python 语言在 Azure Databricks 中开发笔记本和作业。

## <a name="python-apis"></a>Python API

### <a name="pyspark-api"></a>PySpark API

PySpark 是用于 Apache Spark 的 Python API。 这些链接提供了有关 PySpark 的简介和参考。

* [数据帧简介](../spark/latest/dataframes-datasets/introduction-to-dataframes-python.md)
* [结构化流式处理简介](../spark/latest/structured-streaming/demo-notebooks.md)
* [PySpark API 参考](https://spark.apache.org/docs/latest/api/python/pyspark.html)

### <a name="pandas-api-koalas"></a>pandas API (Koalas)

pandas 是一种 Python API，可轻松直观地处理“关系”数据。
Koalas 实现了适用于 Apache Spark 的 pandas 数据帧 API。

* [Koalas](koalas.md)
  * [惠?](koalas.md#requirements)
  * [笔记本](koalas.md#notebook)
  * [资源](koalas.md#resources)

## <a name="visualizations"></a>可视化效果

Azure Databricks Python 笔记本使用 `display` 函数支持各种类型的可视化效果。

* [Python 中的可视化效果](../notebooks/visualizations/index.md#visualizations-in-python)

还可以使用以下第三方库在 Azure Databricks Python 笔记本中创建可视化效果。

* [Bokeh](../notebooks/visualizations/bokeh.md)
* [Matplotlib](../notebooks/visualizations/matplotlib.md)
* [Plotly](../notebooks/visualizations/plotly.md)

## <a name="interoperability"></a>互操作性

这些文章介绍了支持 PySpark 与 pandas 之间的互操作性的功能。

* [优化 PySpark 与 Pandas 数据帧之间的转换](../spark/latest/spark-sql/spark-pandas.md)
* [Pandas 用户定义的函数](../spark/latest/spark-sql/udf-python-pandas.md)
* [pandas 函数 API](../spark/latest/spark-sql/pandas-function-apis.md)

此文介绍支持 Python 与 SQL 之间的互操作性的功能。

* [用户定义的函数](../spark/latest/spark-sql/udf-python.md)

## <a name="tools"></a>工具

除了 Azure Databricks 笔记本以外，还可以使用以下 Python 开发人员工具：

* [包含 Databricks Connect 的 Jupyter](../dev-tools/databricks-connect.md#jupyter)
* [包含 Databricks Connect 的 PyCharm](../dev-tools/databricks-connect.md#pycharm)

## <a name="libraries"></a>库

[Databricks 运行时](../runtime/index.md)包含很多常用库。 你还可以安装其他第三方的或自定义的 Python 库，用于在 Databricks 群集上运行的笔记本和作业。

### <a name="cluster-based-libraries"></a>基于群集的库

基于群集的库可用于在群集上运行的所有笔记本和作业。
若要了解如何安装基于群集的库，请参阅[在群集上安装库](../libraries/cluster-libraries.md#install-libraries)。

### <a name="notebook-scoped-libraries"></a>作用域为笔记本的库

笔记本范围内的库仅适用于安装了它们的笔记本，并且必须为每个会话重新安装。

* 若要了解 Databricks Runtime 6.4 ML 和更高版本以及 Databricks Runtime 7.1 和更高版本中笔记本范围内的库，请参阅[笔记本范围内的 Python 库](../libraries/notebooks-python-libraries.md)。
* 若要了解 Databricks Runtime 7.0 和更低版本中笔记本范围内的库，请参阅[库实用工具](../dev-tools/databricks-utils.md#dbutils-library)。

## <a name="machine-learning"></a>机器学习

有关 Azure Databricks 上的机器学习的常规信息，请参阅[机器学习和深度学习](../applications/machine-learning/index.md)。

若要通过 scikit-learn 库开始使用机器学习，请使用以下笔记本。 它包括数据加载和准备；模型定型、优化和推理；以及使用 [MLflow](../applications/mlflow/index.md) 的模型部署和管理。

[10 分钟教程：Databricks 上使用 scikit-learn 的机器学习](../applications/mlflow/end-to-end-example.md)

## <a name="resources"></a>资源

* [将单节点工作负荷迁移到 Azure Databricks](../migration/single-node.md)
* [知识库](/databricks/kb/python)