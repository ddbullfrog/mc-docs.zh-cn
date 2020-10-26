---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/14/2020
title: 适用于 R 开发人员的 Azure Databricks
description: 了解如何使用 Azure Databricks 中的 SparkR、sparklyr 和 RStudio 来处理 R 中的 Apache Spark。
ms.openlocfilehash: 7a2cfc5999a6004fab0e0b2cbc82ea6856de5ae6
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472741"
---
# <a name="azure-databricks-for-r-developers"></a>适用于 R 开发人员的 Azure Databricks

本部分提供了一个指南，用于指导如何使用 R 语言在 Azure Databricks 中开发笔记本。

## <a name="r-apis"></a>R API

Azure Databricks 支持两个 API，这些 API 为 Apache Spark 提供 R 接口：[SparkR](https://spark.apache.org/docs/latest/sparkr.html) 和 [sparklyr](https://spark.rstudio.com/)。

### <a name="sparkr"></a>SparkR

这些文章提供了有关 SparkR 的简介和参考。

* [SparkR 概述](overview.md)
* [SparkR ML 教程](tutorials/index.md)
* [SparkR 函数参考](sparkr.md)
* [SparkR 1.6](../../1.6/sparkr/index.md)

### <a name="sparklyr"></a>sparklyr

本文介绍 sparklyr。

* [sparklyr](sparklyr.md)

## <a name="visualizations"></a>可视化效果

Azure Databricks R 笔记本使用 `display` 函数支持各种类型的可视化效果。

* [使用 R 进行可视化](../../../notebooks/visualizations/index.md#visualizations-in-r)

## <a name="tools"></a>工具

除了 Azure Databricks 笔记本以外，还可以使用以下 R 开发人员工具：

* [Azure Databricks 上的 RStudio](rstudio.md)
* [Azure Databricks 上的 Shiny](shiny.md)

* 将 SparkR 和 RStudio Desktop 与 [Databricks Connect](../../../dev-tools/databricks-connect.md#sparkr-rstudio) 配合使用。
* 将 sparklyr 和 RStudio Desktop 与 [Databricks Connect](../../../dev-tools/databricks-connect.md#sparklyr-rstudio) 配合使用。

## <a name="resources"></a>资源

* [知识库](https://docs.microsoft.com/azure/databricks/kb/r/)