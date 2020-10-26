---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/07/2020
title: 常见问题解答 (FAQ) - Azure Databricks
description: 了解有关 Delta Lake 的常见问题解答。
ms.openlocfilehash: 0841ffbcd98879e185bb93e127884a77fa2ef80c
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121890"
---
# <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

## <a name="what-is-delta-lake"></a>什么是 Delta Lake？

[Delta Lake](https://delta.io/) 是可提高[数据湖](https://databricks.com/discover/data-lakes/introduction)可靠性的[开源存储层](https://github.com/delta-io/delta)。 Delta Lake 提供 ACID 事务和可缩放的元数据处理，并可以统一流处理和批数据处理。 Delta Lake 在现有 Data Lake 的顶层运行，与 Apache Spark API 完全兼容。

使用 Azure Databricks 上的 Delta Lake 可以根据工作负荷模式配置 Delta Lake，并提供优化的布局和索引来加快交互式查询速度。

## <a name="how-is-delta-lake-related-to-apache-spark"></a>Delta Lake 与 Apache Spark 之间存在何种关系？

Delta Lake 位于 Apache Spark 之上。 格式和计算层有助于简化大数据管道的生成，提高管道的整体效率。

## <a name="what-format-does-delta-lake-use-to-store-data"></a>Delta Lake 使用哪种格式存储数据？

Delta Lake 使用受版本控制的 Parquet 文件将数据存储在云存储空间中。 除了版本之外，Delta Lake 还存储事务日志用于跟踪针对表或 Blob 存储目录的所有提交（这些提交的目的是提供 ACID 事务）。

## <a name="how-can-i-read-and-write-data-with-delta-lake"></a>如何使用 Delta Lake 读取和写入数据？

你可使用自己喜欢的 Apache Spark API 在 Delta Lake 中读取和写入数据。 请参阅[读取表](delta-batch.md#deltadataframereads)和[写入表](delta-batch.md#deltadataframewrites)。

## <a name="where-does-delta-lake-store-the-data"></a>Delta Lake 将数据存储在何处？

写入数据时，你可在云存储空间中指定位置。 Delta Lake 以 Parquet 格式将数据存储在该位置。

## <a name="can-i-stream-data-directly-into-and-from-delta-tables"></a>是否可将数据直接流式传入和流式传出 Delta 表？

是，你可使用结构化流式处理在 Delta 表中直接写入和读取数据。 请参阅[将数据流式传输到 Delta 表](delta-streaming.md#stream-sink)和[从 Delta 表流式传输数据](delta-streaming.md#stream-source)。

## <a name="does-delta-lake-support-writes-or-reads-using-the-spark-streaming-dstream-api"></a>Delta Lake 是否支持使用 Spark Streaming DStream API 写入或读取数据？

Delta 不支持 DStream API。 建议参阅[表流式读取和写入](delta-streaming.md)。

## <a name="when-i-use-delta-lake-will-i-be-able-to-port-my-code-to-other-spark-platforms-easily"></a>使用 Delta Lake 时，是否可以轻松将代码移植到其他 Spark 平台？

是的。 使用 Delta Lake 时，你使用的是开放式 Apache Spark API，因此可轻松地将代码移植到其他 Spark 平台。 若要移植代码，请将 `delta` 格式替换为 `parquet` 格式。

## <a name="how-do-delta-tables-compare-to-hive-serde-tables"></a>增量表与 Hive SerDe 表之间有何差别？

Delta 表的管理程度更高。 特别是，有几个 Delta Lake 替你代管的 Hive SerDe 参数，在任何时候你都不得手动指定它们：

* `ROWFORMAT`
* `SERDE`
* `OUTPUTFORMAT` 和 `INPUTFORMAT`
* `COMPRESSION`
* `STORED AS`

## <a name="what-ddl-and-dml-features-does-delta-lake-not-support"></a>Delta Lake 不支持哪些 DDL 和 DML 功能？

* 不支持的 DDL 功能：
  * `ANALYZE TABLE PARTITION`
  * `ALTER TABLE [ADD|DROP] PARTITION`
  * `ALTER TABLE RECOVER PARTITIONS`
  * `ALTER TABLE SET SERDEPROPERTIES`
  * `CREATE TABLE LIKE`
  * `INSERT OVERWRITE DIRECTORY`
  * `LOAD DATA`
* 不支持的 DML 功能：
  * 具有静态分区的 `INSERT INTO [OVERWRITE]` 表
  * `INSERT OVERWRITE TABLE`，用于具有动态分区的表
  * 分桶
  * 从表中读取时指定架构
  * 在 `TRUNCATE TABLE` 中使用 `PARTITION (part_spec)` 指定目标分区

## <a name="does-delta-lake-support-multi-table-transactions"></a>Delta Lake 是否支持多表事务？

Delta Lake 不支持多表事务和外键。 Delta Lake 支持 table 级别的事务。

## <a name="how-can-i-change-the-type-of-a-column"></a>如何更改列的类型？

更改列的类型或删除列需要重写该表。 有关示例，请参阅[更改列类型](delta-batch.md#change-column-type)。

## <a name="what-does-it-mean-that-delta-lake-supports-multi-cluster-writes"></a><a id="multi-cluster"> </a><a id="what-does-it-mean-that-delta-lake-supports-multi-cluster-writes"> </a>Delta Lake 支持多群集写入是什么意思？

这意味着 Delta Lake 会锁定，确保同时从多个群集写入表的查询不会导致该表损坏。 但这并不意味着，如果发生写入冲突（例如更新和删除相同的内容），它们都将成功。 相反，其中一个写入操作将失败（以原子方式），并且错误会让你重试该操作。

## <a name="can-i-modify-a-delta-table-from-different-workspaces"></a>是否可从不同的工作区修改增量表？

是，你可同时从不同的工作区修改同一个 Delta 表。 此外，如果从某个工作区编写一个进程，则其他工作区中的读者将看到一致的视图。

## <a name="can-i-access-delta-tables-outside-of-databricks-runtime"></a>是否可以在 Databricks Runtime 的外部访问增量表？

需要考虑两种情况：外部写入和外部读取。

* 外部写入：Delta Lake 以事务日志的形式维护其他元数据，从而为读者启用 ACID 事务和快照隔离。 若要确保正确更新事务日志并执行正确的验证，必须通过 Databricks Runtime 执行写入操作。

* 外部读取：Delta 表存储以开放格式 (Parquet) 编码的数据，使其他可读懂此格式的工具能够读取数据。 但是，由于其他工具不支持 Delta Lake 事务日志，因此它们可能会错误地读取过时的已删除的数据、未提交的数据或失败事务的部分结果。

  如果数据是静态的（即没有活动的作业写入表），可使用保留期为 `ZERO HOURS` 的 `VACUUM` 清理当前不属于表的任何过时的 Parquet 文件。 此操作会将 DBFS 中的 Parquet 文件设置为一致的状态，使外部工具现在可读取它们。

  但是，Delta Lake 依赖于过时的快照来实现以下功能，因此在使用允许零保留期的 `VACUUM` 时，会发生中断：

  * 针对读者隔离快照 - 长时间运行的作业将继续从作业开始的那一刻读取一致的快照，即使同时在对该表进行修改也是如此。 如果运行的 `VACUUM` 的保留期小于这些作业的长度，可能会导致它们失败并出现 `FileNotFoundException`。
  * 从 Delta 表流式传输 - 从原始文件读取的流写入到表中，以确保恰好处理一次。 结合 `OPTIMIZE` 使用时，具有零保留期的 `VACUUM` 可能会在流有时间处理这些文件之前将它们删除，从而导致其失败。

  为此，建议仅对必须由外部工具读取的静态数据集使用以上方法。