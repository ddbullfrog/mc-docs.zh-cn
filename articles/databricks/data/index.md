---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 数据 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用数据。
ms.openlocfilehash: 1fd3c8f9a07cc8d49f81650fdc8a2658f03ba98a
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121951"
---
# <a name="data"></a>数据

此部分介绍如何在 Azure Databricks 中使用数据。 可以：

* 直接从导入的数据创建表。 表架构存储在默认 Azure Databricks 内部元存储中，你也可以配置和使用外部元存储。
* 使用各种 Apache Spark 数据源。
* 将数据导入到 Databricks 文件系统 (DBFS)（一个已装载到 Azure Databricks 工作区并在 Azure Databricks 群集上可用的分布式文件系统），并使用 [DBFS CLI](../dev-tools/cli/dbfs-cli.md)、[DBFS API](../dev-tools/api/latest/dbfs.md)、[Databricks 文件系统实用工具 (dbutils.fs)](../dev-tools/databricks-utils.md#dbutils-fs)、[Spark API](databricks-file-system.md#dbfs-spark) 和[本地文件 API](databricks-file-system.md#fuse) 访问数据。

本部分的内容：

* [数据概述](data.md)
* [数据库和表](tables.md)
* [Metastores](metastores/index.md)（元存储）
* [数据源](data-sources/index.md)
* [Databricks 文件系统 (DBFS)](databricks-file-system.md)
* [Azure Databricks 数据集](databricks-datasets.md)
* [FileStore](filestore.md)