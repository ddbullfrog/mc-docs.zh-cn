---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/21/2020
title: 通过 DBIO 向云存储进行事务性写入 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Databricks DBIO 包对云存储执行事务性写入。
ms.openlocfilehash: 0e0c46b4dfe9bd41fb40494d84d250bbbaea1182
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473044"
---
# <a name="transactional-writes-to-cloud-storage-with-dbio"></a><a id="dbio-commit"> </a><a id="transactional-writes-to-cloud-storage-with-dbio"> </a>通过 DBIO 向云存储进行事务性写入

Databricks DBIO 包为 Apache Spark 作业提供了对云存储的事务性写入。 这解决了在云原生设置中使用 Spark（例如，直接写入存储服务）时发生的许多性能和正确性问题。

使用 DBIO 事务提交，以 `_started_<id>` 和 `_committed_<id>` 开头的元数据文件将伴随着 Spark 作业创建的数据文件。 通常不应直接更改这些文件， 而应使用 `VACUUM` 命令将其清除。

## <a name="clean-up-uncommitted-files"></a><a id="clean-up-uncommitted-files"> </a><a id="vacuum-spark"> </a>清除未提交的文件

若要清除 Spark 作业遗留的未提交文件，请使用 `VACUUM` 命令将其删除。 通常，`VACUUM` 在 Spark 作业完成后自动执行，但如果作业中止，你也可以手动运行该命令。

例如，`VACUUM ... RETAIN 1 HOUR` 会删除一小时前的未提交文件。

> [!IMPORTANT]
>
> * 请避免在不到一小时的时间内执行 vacuum 命令。 这可能会导致数据不一致。

另请参阅 [Vacuum](language-manual/vacuum.md)。

### <a name="sql"></a>SQL

```sql
-- recursively vacuum an output path
VACUUM '/path/to/output/directory' [RETAIN <N> HOURS]

-- vacuum all partitions of a catalog table
VACUUM tableName [RETAIN <N> HOURS]
```

### <a name="scala"></a>Scala

```scala
// recursively vacuum an output path
spark.sql("VACUUM '/path/to/output/directory' [RETAIN <N> HOURS]")

// vacuum all partitions of a catalog table
spark.sql("VACUUM tableName [RETAIN <N> HOURS]")
```