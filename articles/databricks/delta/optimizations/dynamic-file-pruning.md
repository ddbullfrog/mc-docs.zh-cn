---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/31/2020
title: 动态文件修剪 - Azure Databricks
description: 了解 Azure Databricks 上的 Delta Lake 如何改进对 Delta 表的查询。
ms.openlocfilehash: 9d02cdb7eda5495c8d2f1d2f9001c80640d9a2e6
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121875"
---
# <a name="dynamic-file-pruning"></a>动态文件修剪

动态文件修剪 (DFP) 可以显著提高 Delta 表上许多查询的性能。 DFP 对于非分区表或非分区列上的联接特别有效。 DFP 的性能影响通常与数据聚类相关，因此请考虑使用 [Z 排序](file-mgmt.md#delta-zorder)以最大限度地提高 DFP 的效益。

有关 DFP 的背景和用例，请参阅[通过动态文件修剪在 Delta Lake 上更快进行 SQL 查询](https://databricks.com/blog/2020/04/30/faster-sql-queries-on-delta-lake-with-dynamic-file-pruning.html)。

> [!NOTE]
>
> 在 Databricks Runtime 6.1 及更高版本中可用。

DFP 由以下 Apache Spark 配置选项控制：

* `spark.databricks.optimizer.dynamicPartitionPruning`（默认值为 `true`）：指示优化器向下推 DFP 筛选器的主标志。 设置为 `false` 时，DFP 将不起作用。
* `spark.databricks.optimizer.deltaTableSizeThreshold`（默认值为 `10,000,000,000 bytes (10 GB)`）：表示连接探测侧触发 DFP 所需的 Delta 表的最小大小（以字节为单位）。 如果探测侧不是很大，那么向下推筛选器可能不值得，我们可以简单地扫描整个表。 可以通过运行 `DESCRIBE DETAIL table_name` 命令然后查看 `sizeInBytes` 列来查找 Delta 表的大小。
* `spark.databricks.optimizer.deltaTableFilesThreshold`（默认值为 `1000`）：表示连接探测侧触发 DFP 所需的 Delta 表的文件数。 如果探测侧表包含的文件少于阈值，则不会触发 DPP。 如果表只有几个文件，则启用 DFP 可能不值得。 可以通过运行 `DESCRIBE DETAIL table_name` 命令然后查看 `numFiles` 列来查找 Delta 表的大小。