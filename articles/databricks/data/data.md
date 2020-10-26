---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: 数据概述 - Azure Databricks
description: 了解如何使用 Apache Spark 和本地 API 导入和读取数据，以及如何使用 Azure Databricks 中的 DBFS 命令编辑和删除数据。
ms.openlocfilehash: 8202c9a7b019471bbb1cf4bfaf2a63a47bcf4409
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121931"
---
# <a name="data-overview"></a><a id="access-data"> </a><a id="data-overview"> </a>数据概述

本文介绍如何使用 UI 将数据导入 Azure Databricks，使用 Spark 和本地 API 读取导入的数据，以及如何使用 [Databricks 文件系统 (DBFS)](databricks-file-system.md) 命令修改导入的数据。

## <a name="import-data"></a>导入数据

如果本地计算机上有要使用 Azure Databricks 进行分析的小型数据文件，可使用 UI 将其导入 DBFS。

> [!NOTE]
>
> 管理员用户可能会禁用此功能。 若要启用或禁用此设置，请参阅[管理数据上传](../administration-guide/workspace/dbfs-ui-upload.md)。

可通过两种方法，使用 UI 将数据上传到 DBFS：

* 在[上传数据 UI](databricks-file-system.md#user-interface) 中将文件上传到 FileStore。

  > [!div class="mx-imgBorder"]
  > ![上传数据](../_static/images/notebooks/upload-data-upload-step.png)

* 使用[创建表 UI](tables.md#create-table-ui) 将数据上传到[表](tables.md)，也可通过登陆页面上的“导入和浏览数据”框访问数据。

  > [!div class="mx-imgBorder"]
  > ![导入和浏览数据](../_static/images/data-import/import-landing.png)

使用这些方法导入到 DBFS 的文件存储在 [FileStore](filestore.md#filestore) 中。

对于生产环境，建议使用 [DBFS CLI](../dev-tools/cli/dbfs-cli.md)、[DBFS API](../dev-tools/api/latest/dbfs.md) 和 [Databricks 文件系统实用程序 (dbutils.fs)](../dev-tools/databricks-utils.md#dbutils-fs) 将文件显式上传到 DBFS。

还可使用各种[数据源](data-sources/index.md#data-sources)来访问数据。

## <a name="read-data-on-cluster-nodes-using-spark-apis"></a>使用 Spark API 读取群集节点上的数据

使用 [Spark API](../getting-started/spark/dataframes.md#spark-dataframes) 将导入到 DBFS 的数据读取到 Apache Spark DataFrames 中。 例如，如果导入 CSV 文件，可使用以下示例之一读取数据。

> [!TIP]
>
> 为了简化访问，建议创建一个表。 有关详细信息，请参阅[数据库和表](tables.md#tables)。

### <a name="python"></a>Python

```python
sparkDF = spark.read.csv('/FileStore/tables/state_income-9f7c5.csv', header="true", inferSchema="true")
```

### <a name="r"></a>R

```r
sparkDF <- read.df(source = "csv", path = "/FileStore/tables/state_income-9f7c5.csv", header="true", inferSchema = "true")
```

### <a name="scala"></a>Scala

```scala
val sparkDF = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/FileStore/tables/state_income-9f7c5.csv")
```

## <a name="read-data-on-cluster-nodes-using-local-apis"></a>使用本地 API 读取群集节点上的数据

还可使用[本地文件 API](databricks-file-system.md#fuse) 在 Spark 驱动程序节点上运行的程序中读取导入到 DBFS 的数据。 例如： 。

### <a name="python"></a>Python

```python
pandas_df = pd.read_csv("/dbfs/FileStore/tables/state_income-9f7c5.csv", header='infer')
```

### <a name="r"></a>R

```r
df = read.csv("/dbfs/FileStore/tables/state_income-9f7c5.csv", header = TRUE)
```

## <a name="modify-uploaded-data"></a>修改已上传的数据

不能直接在 Azure Databricks 中编辑导入的数据，但可使用 [Spark API](../getting-started/spark/dataframes.md#spark-dataframes)、[DBFS CLI](../dev-tools/cli/dbfs-cli.md)、[DBFS API](../dev-tools/api/latest/dbfs.md) 和 [Databricks 文件系统实用程序 (dbutils.fs)](../dev-tools/databricks-utils.md#dbutils-fs) 覆盖数据文件。

若要从 DBFS 删除数据，请使用上述 API 和工具。 例如，可使用 [Databricks 实用程序](../dev-tools/databricks-utils.md)命令 `dbutils.fs.rm`：

```python
dbutils.fs.rm("dbfs:/FileStore/tables/state_income-9f7c5.csv", true)
```

> [!WARNING]
>
> 无法恢复已删除的数据。