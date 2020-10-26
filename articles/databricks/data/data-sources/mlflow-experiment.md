---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/01/2020
title: MLflow 试验 - Azure Databricks
description: 了解如何使用 Azure Databricks 加载 MLflow 试验运行数据。
ms.openlocfilehash: 4922eeb4be1d8aab9d767142590524066c76a457
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121846"
---
# <a name="mlflow-experiment"></a><a id="mlflow-exp-datasource"> </a><a id="mlflow-experiment"> </a>MLflow 试验

MLflow 试验数据源提供了一种用于加载 MLflow 试验运行数据的标准 API。
可以从[笔记本试验](../../applications/mlflow/tracking.md#experiments)加载数据，也可以使用 MLflow 试验名称或试验 ID 加载数据。

## <a name="requirements"></a>要求

Databricks Runtime 6.0 ML 或更高版本。

## <a name="load-data-from-the-notebook-experiment"></a>从笔记本试验加载数据

若要从笔记本试验加载数据，请使用 `load()`。

### <a name="python"></a>Python

```python
df = spark.read.format("mlflow-experiment").load()
display(df)
```

### <a name="scala"></a>Scala

```scala
val df = spark.read.format("mlflow-experiment").load()
display(df)
```

## <a name="load-data-using-experiment-ids"></a>使用试验 ID 加载数据

若要从一个或多个工作区试验加载数据，请按如下所示指定试验 ID。

### <a name="python"></a>Python

```python
df = spark.read.format("mlflow-experiment").load("3270527066281272")
display(df)
```

### <a name="scala"></a>Scala

```scala
val df = spark.read.format("mlflow-experiment").load("3270527066281272,953590262154175")
display(df)
```

## <a name="load-data-using-experiment-name"></a>使用试验名称加载数据

还可将试验名称传递给 `load()` 方法。

### <a name="python"></a>Python

```python
expId = mlflow.get_experiment_by_name("/Shared/diabetes_experiment/").experiment_id
df = spark.read.format("mlflow-experiment").load(expId)
display(df)
```

### <a name="scala"></a>Scala

```scala
val expId = mlflow.getExperimentByName("/Shared/diabetes_experiment/").get.getExperimentId
val df = spark.read.format("mlflow-experiment").load(expId)
display(df)
```

## <a name="filter-data-based-on-metrics-and-parameters"></a>基于指标和参数筛选数据

本部分的示例演示如何在从试验中加载数据后对数据进行筛选。

### <a name="python"></a>Python

```python
df = spark.read.format("mlflow-experiment").load("3270527066281272")
filtered_df = df.filter("metrics.loss < 0.01 AND params.learning_rate > '0.001'")
display(filtered_df)
```

### <a name="scala"></a>Scala

```scala
val df = spark.read.format("mlflow-experiment").load("3270527066281272")
val filtered_df = df.filter("metrics.loss < 1.85 AND params.num_epochs > '30'")
display(filtered_df)
```

## <a name="schema"></a>架构

数据源返回的数据帧的架构为：

```
root
|-- run_id: string
|-- experiment_id: string
|-- metrics: map
|    |-- key: string
|    |-- value: double
|-- params: map
|    |-- key: string
|    |-- value: string
|-- tags: map
|    |-- key: string
|    |-- value: string
|-- start_time: timestamp
|-- end_time: timestamp
|-- status: string
|-- artifact_uri: string
```