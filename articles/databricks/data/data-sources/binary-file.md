---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 二进制文件 - Azure Databricks
description: 了解如何使用 Azure Databricks 读取二进制文件中的数据。
ms.openlocfilehash: 1be18946fa789a0dae6ce6b9fe24f6e0b793b615
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121822"
---
# <a name="binary-file"></a>二进制文件

Databricks Runtime 支持[二进制文件](https://spark.apache.org/docs/latest/sql-data-sources-binaryFile.html)数据源，该数据源读取二进制文件并将每个文件转换为包含该文件的原始内容和元数据的单个记录。 二进制文件数据源会生成一个包含以下列和可能的分区列的数据帧：

若要读取二进制文件，请将数据源 `format` 指定为 `binaryFile`。

## <a name="options"></a>选项

若要加载其路径与给定 glob 模式匹配的文件，同时保留分区发现行为，可以使用 `pathGlobFilter` 选项。 以下代码使用分区发现从输入目录读取所有 JPG 文件：

```python
df = spark.read.format("binaryFile").option("pathGlobFilter", "*.jpg").load("<path-to-dir>")
```

如果要忽略分区发现并以递归方式搜索输入目录下的文件，请使用 `recursiveFileLookup` 选项。 此选项会搜索整个嵌套目录，即使这些目录的名称不遵循 `date=2019-07-01` 之类的分区命名方案。
以下代码从输入目录中以递归方式读取所有 JPG 文件，并忽略分区发现：

```python
df = spark.read.format("binaryFile") \
  .option("pathGlobFilter", "*.jpg") \
  .option("recursiveFileLookup", "true") \
  .load("<path-to-dir>")
```

Scala、Java 和 R 存在类似的 API。

> [!NOTE]
>
> 若要在重新加载数据时提高读取性能，Azure Databricks 建议你在保存从二进制文件加载的数据时禁用压缩：
>
> ```python
> spark.conf.set("spark.sql.parquet.compression.codec", "uncompressed")
> df.write.format("delta").save("<path-to-table>")
> ```