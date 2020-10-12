---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: Databricks 运行时 - Azure Databricks
description: 了解 Databricks 运行时的类型和运行时内容。
ms.openlocfilehash: 0288479e95888988037a943e67dbcfea52f5ffc6
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937725"
---
# <a name="databricks-runtimes"></a><a id="databricks-runtimes"> </a><a id="dbr-overview"> </a>Databricks 运行时

Databricks 运行时是在 Azure Databricks [群集](../clusters/index.md)上运行的核心组件集。 Azure Databricks 提供多种类型的运行时。

* [Databricks Runtime](dbr.md)

  Databricks Runtime 包括 Apache Spark，但还添加了许多可以显著提高大数据分析可用性、性能和安全性的组件与更新。

* [用于机器学习的 Databricks Runtime](mlruntime.md)

  Databricks Runtime ML 是 Databricks Runtime 的变体，其中添加了多个常用机器学习库（包括 TensorFlow、Keras、PyTorch 和 XGBoost）。

* [用于基因组学的 Databricks Runtime](genomicsruntime.md)

  用于基因组学的 Databricks Runtime 是 Databricks Runtime 的变体，已针对处理基因组和生物医学数据而进行了优化。

* [Databricks Light](light.md)

  Databricks Light 为不需要由 Databricks Runtime 提供的高级性能、可靠性或自动缩放优势的作业提供了运行时选项。

创建群集时，可以从受支持的运行时版本中进行选择。

> [!div class="mx-imgBorder"]
> ![选择 Databricks 运行时](../_static/images/clusters/runtime-version.png)

有关每个运行时变体的内容的信息，请参阅[发行说明](../release-notes/runtime/releases.md)。