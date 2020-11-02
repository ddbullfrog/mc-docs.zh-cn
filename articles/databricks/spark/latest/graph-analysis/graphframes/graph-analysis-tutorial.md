---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/21/2020
title: 使用 GraphFrames 执行图分析的教程 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 GraphFrames 执行图分析。
ms.openlocfilehash: 7c23377cd7950724bfb9c03e1be0f4e6d684e285
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472897"
---
# <a name="graph-analysis-tutorial-with-graphframes"></a>使用 GraphFrames 执行图分析的教程

本教程笔记本介绍如何使用 GraphFrames 执行图分析。 Databricks 建议使用运行[用于机器学习的 Databricks Runtime](../../../../runtime/mlruntime.md) 的群集，因为它包括 GraphFrames 的优化安装。

运行笔记本：

1. 如果未使用运行 Databricks Runtime ML 的群集，请使用[这些方法](../../../../libraries/index.md)中的一种安装 [GraphFrames 库](https://spark-packages.org/package/graphframes/graphframes)。
2. 从 Kaggle 下载旧金山湾区共享单车[数据](https://www.kaggle.com/benhamner/sf-bay-area-bike-share)，并将其解压。 必须使用第三方身份验证登录 Kaggle，或创建 Kaggle 帐户并登录。
3. 使用 [创建表 UI](../../../../data/tables.md#create-table-ui) 上传 `station.csv` 和 `trip.csv`。

   这些表名为 `station_csv` 和 `trip_csv`。

## <a name="graph-analysis-with-graphframes-notebook"></a>使用 GraphFrames 笔记本执行图分析

[获取笔记本](../../../../_static/notebooks/graph-analysis-graphframes.html)