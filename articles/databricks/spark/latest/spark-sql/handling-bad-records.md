---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: 处理错误的记录和文件 - Azure Databricks
description: 了解在 Azure Databricks 中使用 Apache Spark SQL 查询时如何处理错误的记录和文件。
ms.openlocfilehash: 26a3500036a093ccbb781211027467436fc40881
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472818"
---
# <a name="handling-bad-records-and-files"></a><a id="handling-bad-records-and-files"> </a><a id="spark-sql-bad-records"> </a>处理错误的记录和文件

在从基于文件的数据源读取数据时，Apache Spark SQL 会面临两种典型的错误案例。 第一种，这些文件可能是无法读取的（例如，它们可能已缺失、不可访问或已损坏）。 第二种，即使这些文件是可处理的，也有可能无法分析某些记录（例如，由于语法错误和架构不匹配）。

Azure Databricks 提供了一个统一接口，以便处理错误的记录和文件，而无需中断 Spark 作业。 通过设置数据源选项 `badRecordsPath`，可从异常日志中获取异常记录/文件和原因。 `badRecordsPath` 会指定一个路径来存储用于记录信息的异常文件，这些信息与 CSV 和 JSON 源的错误记录以及所有基于文件的内置源（例如 Parquet）的错误文件相关。

另外，在读取文件时可能会出现暂时性错误，如网络连接异常、IO 异常，等等。 这些错误会被忽略，也会记录在 `badRecordsPath` 下，并且 Spark 将会继续运行任务。

> [!NOTE]
>
> 配合 Delta Lake 使用的 `badRecordsPath` 数据源具有以下几个重要限制：
>
> * 它是非事务性的，可能会导致不一致的结果。
> * Delta Lake 会将暂时性错误视为失败。

## <a name="examples"></a>示例

### <a name="unable-to-find-input-file"></a>无法找到输入文件

```scala
val df = spark.read
  .option("badRecordsPath", "/tmp/badRecordsPath")
  .parquet("/input/parquetFile")

// Delete the input parquet file '/input/parquetFile'
dbutils.fs.rm("/input/parquetFile")

df.show()
```

在上面的示例中，由于 `df.show()` 无法找到输入文件，因此 Spark 会创建一个 JSON 格式的异常文件来记录该错误。 例如，`/tmp/badRecordsPath/20170724T101153/bad_files/xyz` 是异常文件的路径。 此文件位于指定的 `badRecordsPath` 目录 `/tmp/badRecordsPath` 下。 `20170724T101153` 是此 `DataFrameReader` 的创建时间。 `bad_files` 是异常类型。 `xyz` 是包含 JSON 记录的文件，其中有错误文件的路径和异常/原因消息。

### <a name="input-file-contains-bad-record"></a>输入文件包含错误记录

```scala
// Creates a json file containing both parsable and corrupted records
Seq("""{"a": 1, "b": 2}""", """{bad-record""").toDF().write.text("/tmp/input/jsonFile")

val df = spark.read
  .option("badRecordsPath", "/tmp/badRecordsPath")
  .schema("a int, b int")
  .json("/tmp/input/jsonFile")

df.show()
```

在此示例中，数据帧只包含第一个可分析的记录 (`{"a": 1, "b": 2}`)。 第二个错误记录 (`{bad-record`) 记录在异常文件中，该文件是 JSON 文件，位于 `/tmp/badRecordsPath/20170724T114715/bad_records/xyz` 中。 该异常文件包含错误记录、包含此记录的文件的路径，以及异常/原因消息。 找到异常文件后，可以使用 JSON 读取器来处理这些文件。