---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/22/2020
title: Koalas - Azure Databricks
description: 了解如何使用 Koalas API 访问 Azure Databricks 中的数据。
ms.openlocfilehash: 6c1ec2ab7f34710c2904e73cc7208a4b7d9d8e9d
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937627"
---
# <a name="koalas"></a>Koalas

[Koalas](https://github.com/databricks/koalas) 允许你使用 [pandas](https://pandas.pydata.org/) 数据帧 API 来访问 Apache Spark 中的数据。

## <a name="requirements"></a>要求

在 Databricks Runtime 7.0 或更低版本上，将 Koalas 作为 Azure Databricks [PyPI 库](../libraries/workspace-libraries.md#pypi-libraries)进行安装。

## <a name="notebook"></a>笔记本

以下笔记本演示如何从 pandas 迁移到 Koalas。

### <a name="pandas-to-koalas-notebook"></a>pandas 到 Koalas 笔记本

[获取笔记本](../_static/notebooks/pandas-to-koalas-in-10-minutes.html)

## <a name="resources"></a>资源

* [Koalas 文档](https://koalas.readthedocs.io/en/latest/index.html)
* [在 Apache Spark 上从 pandas 迁移到 Koalas 只需 10 分钟](https://databricks.com/blog/2020/03/31/10-minutes-from-pandas-to-koalas-on-apache-spark.html)