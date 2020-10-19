---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/22/2020
title: 适用于 Scala 开发人员的 Azure Databricks - Azure Databricks
description: 了解有关使用 Scala 在 Azure Databricks 中进行开发的信息。
ms.openlocfilehash: 1cae139b017377bdf5b15f1f5e44571555f0b23e
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937830"
---
# <a name="azure-databricks-for-scala-developers"></a>适用于 Scala 开发人员的 Azure Databricks

本部分提供了一个指南，用于指导如何使用 Scala 语言在 Azure Databricks 中开发笔记本和作业。

## <a name="scala-api"></a>Scala API

这些链接提供对 Apache Spark Scala API 的介绍和参考。

* [数据帧简介](../spark/latest/dataframes-datasets/introduction-to-dataframes-scala.md)
  * [复杂数据和嵌套数据](../spark/latest/dataframes-datasets/complex-nested-data.md)
  * [聚合器](../spark/latest/dataframes-datasets/aggregators.md)
* [数据集简介](../spark/latest/dataframes-datasets/introduction-to-datasets.md)
* [结构化流式处理简介](../spark/latest/structured-streaming/demo-notebooks.md)
* [Apache Spark API 参考](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.package)

## <a name="visualizations"></a>可视化效果

Azure Databricks Scala 笔记本使用 `display` 函数支持各种类型的可视化效果。

* [可视化概述](../notebooks/visualizations/index.md#visualizations-in-scala)
* [深入探讨使用 Scala 进行可视化](../notebooks/visualizations/charts-and-graphs-scala.md)

## <a name="interoperability"></a>互操作性

此部分介绍支持 Scala 与 SQL 之间互操作性的功能。

* [用户定义的函数](../spark/latest/spark-sql/udf-scala.md)
* [用户定义聚合函数](../spark/latest/spark-sql/udaf-scala.md)

## <a name="tools"></a>工具

除了 Azure Databricks 笔记本以外，还可以使用以下 Scala 开发人员工具

* [使用 Databricks 连接的 IntelliJ](../dev-tools/databricks-connect.md#intellij-scala-or-java)

## <a name="libraries"></a>库

[Databricks 运行时](../runtime/index.md)提供许多库。 若要将第三方或本地生成的 Scala 库提供给 Azure Databricks 群集上运行的笔记本和作业，可按照以下说明安装库：

* [在群集中安装 Scala 库](../libraries/workspace-libraries.md#maven-or-spark-package)

## <a name="resources"></a>资源

* [知识库](/databricks/kb/scala)