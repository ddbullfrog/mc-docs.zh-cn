---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/17/2020
title: GraphX（旧版）图分析教程 - Azure Databricks
description: 了解如何使用 GraphX 在 Azure Databricks 中执行图分析。
ms.openlocfilehash: 4faabb138405ace5e2907a788ee588f3304b226f
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472845"
---
# <a name="graph-analysis-tutorial-with-graphx-legacy"></a><a id="graph-analysis-tutorial-with-graphx-legacy"> </a><a id="graphx"> </a>GraphX（旧版）图分析教程

本教程笔记本介绍如何使用 GraphX 执行图分析。 若要运行笔记本：

1. 从 Kaggle 下载 SF Bay Area Bike Share [数据](https://www.kaggle.com/benhamner/sf-bay-area-bike-share)，并将其解压缩。 必须使用第三方身份验证登录到 Kaggle，或创建 Kaggle 帐户并登录到该帐户。
2. 使用[创建表 UI](../../../data/tables.md#create-table-ui) 上传 `station.csv` 和 `trip.csv`。

   这些表名为 `station_csv` 和 `trip_csv`。

## <a name="graph-analysis-with-graphx-notebook"></a>使用 GraphX 笔记本执行图分析

[获取笔记本](../../../_static/notebooks/graph-analysis-graphx.html)