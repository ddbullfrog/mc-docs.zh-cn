---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/18/2020
title: Parquet 文件 - Azure Databricks
description: 了解如何使用 Azure Databricks 从 Apache Parquet 文件中读取数据。
ms.openlocfilehash: 447bc7b557579c7c79a03694b2513b591abc3adf
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121916"
---
# <a name="parquet-file"></a>Parquet 文件

[Apache Parquet](https://parquet.apache.org/) 是一种列式文件格式，可提供优化来提高查询速度，比 [CSV](read-csv.md) 或 [JSON](read-json.md) 文件格式要高效得多。

如需了解详情，请参阅 [Parquet 文件](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html)。

## <a name="options"></a>选项

有关支持的读取和写入选项，请参阅以下 Apache Spark 参考文章。

* 读取
  * [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=dataframereader#pyspark.sql.DataFrameReader.parquet)
  * [Scala](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/DataFrameReader.html#parquet(paths:String*):org.apache.spark.sql.DataFrame)
* 写入
  * [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=dataframereader#pyspark.sql.DataFrameWriter.parquet)
  * [Scala](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/DataFrameWriter.html#parquet(path:String):Unit)

以下笔记本显示了如何在 Parquet 文件中读取和写入数据。

### <a name="reading-parquet-files-notebook"></a>读取 Parquet 文件笔记本

[获取笔记本](../../_static/notebooks/read-parquet-files.html)