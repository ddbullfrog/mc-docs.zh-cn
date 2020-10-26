---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 通过缓存优化性能 - Azure Databricks
description: 了解增量缓存如何通过使用快速中间数据格式在节点的本地存储中创建远程文件的副本来加快数据读取。
ms.openlocfilehash: f9e00f99dc961c32ec5a5775bfbf153e86da2e46
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121877"
---
# <a name="optimize-performance-with-caching"></a>通过缓存优化性能

增量缓存通过使用快速中间数据格式在节点的本地存储中创建远程文件的副本来加快数据读取。 必须从远程位置提取文件时，会自动缓存数据。 然后在本地连续读取上述数据，这会显著提高读取速度。

增量缓存支持读取 DBFS、HDFS、Azure Blob 存储、Azure Data Lake Storage Gen1 和 Azure Data Lake Storage Gen2 中的 Parquet 文件。 它不支持其他存储格式，例如 CSV、JSON 和 ORC。

> [!NOTE]
>
> 增量缓存适用于所有 Parquet 文件，并且不限于 [Delta Lake 格式的文件](../delta-faq.md)。

## <a name="delta-and-apache-spark-caching"></a><a id="delta-and-apache-spark-caching"> </a><a id="delta-and-rdd-cache-comparison"> </a>增量缓存和 Apache Spark 缓存

Azure Databricks 中提供了两种类型的缓存：增量缓存和 Apache Spark 缓存。 以下是每个类型的特征：

* **存储数据的类型** ：增量缓存包含远程数据的本地副本。 它可以提高各种查询的性能，但不能用于存储任意子查询的结果。 Spark 缓存可以存储任何子查询数据的结果，以及以非 Parquet 格式（例如 CSV、JSON 和 ORC）存储的数据。
* **性能** ：与 Spark 缓存中的数据相比，读取和操作增量缓存中的数据速度更快。 原因在于，增量缓存使用的是高效的解压缩算法，并以最佳格式输出数据，从而使用全阶段代码生成进行进一步处理。
* **自动和手动控制** ：启用增量缓存后，必须从远程源提取的数据会自动添加到该缓存。 此过程完全透明，无需任何操作。 但若要预先将数据预加载到缓存，可以使用 `CACHE` 命令（请参阅[缓存数据子集](#cache-a-subset-of-the-data)）。 使用 Spark 缓存时，必须手动指定要缓存的表和查询。
* **磁盘和基于内存** ：增量缓存完全存储在本地磁盘上，因此 Spark 中的其他操作不会占用内存。 由于新式 SSD 读取速度较快，因此增量缓存可以完全驻留于磁盘，并且不会对其性能产生负面影响。 相反，Spark 缓存使用内存。

> [!NOTE]
>
> 可以同时使用增量缓存和 Apache Spark 缓存。

### <a name="summary"></a>总结

下表总结了增量缓存和 Apache Spark 缓存之间的主要区别，以便选择最适合工作流的工具：

| 功能                           | 增量缓存                                                                          | Apache Spark 缓存                                             |
|-----------------------------------|--------------------------------------------------------------------------------------|----------------------------------------------------------------|
| 存储格式                         | 工作器节点上的本地文件。                                                        | 内存中块，但取决于存储级别。             |
| 适用对象                        | WASB 和其他文件系统上存储的任何 Parquet 表格。                             | 任何 RDD 或数据帧。                                          |
| 触发                         | 自动执行，第一次读取时（如果启用了缓存）。                              | 手动执行，需要更改代码。                               |
| 已评估                         | 惰性。                                                                              | 惰性。                                                        |
| 强制缓存                       | `CACHE` 和 `SELECT`                                                                 | `.cache` + 任何实现缓存的操作和 `.persist`。 |
| 可用性                      | 可以使用配置标志启用或禁用，可以在某些节点类型上禁用。 | 始终可用。                                              |
| 逐出                           | 更改任何文件时自动执行，重启群集时手动执行。                | 以 LRU 方式自动执行，使用 `unpersist` 手动执行。       |

## <a name="delta-cache-consistency"></a>增量缓存一致性

增量缓存会自动检测创建或删除数据文件的时间，并相应地更新其内容。 可以写入、修改和删除表格数据，并且无需显式地使缓存数据无效。

增量缓存会自动检测缓存后修改或覆盖的文件。 所有过时项都将自动失效，并从缓存中逐出。

## <a name="use-delta-caching"></a><a id="use-delta-caching"> </a><a id="worker-instance-type"> </a>使用增量缓存

若要使用增量缓存，请在配置群集时选择“增量缓存加速”工作器类型。

> [!div class="mx-imgBorder"]
> ![缓存加速群集](../../_static/images/delta/delta-cache-cluster-creation-azure.png)

默认启用增量缓存，并配置为最多使用工作器节点随附的本地 SSD 的一半可用空间。

有关配置选项，请参阅[配置增量缓存](#configure-cache)。

## <a name="cache-a-subset-of-the-data"></a>缓存一部分数据

若要显式选择要缓存的数据子集，请使用以下语法：

```sql
CACHE SELECT column_name[, column_name, ...] FROM [db_name.]table_name [ WHERE boolean_expression ]
```

无需使用此命令即可正常使用增量缓存（首次访问时会自动缓存数据）。 但如果需要查询性能保持一致，它可能会有所帮助。

有关示例和详细信息，请参阅[缓存](../../spark/latest/spark-sql/language-manual/cache-dbio.md)。

## <a name="monitor-the-delta-cache"></a>监视增量缓存

可以在 Spark UI 的“存储”选项卡中检查每个执行程序上的增量缓存的当前状态。

> [!div class="mx-imgBorder"]
> ![监视增量缓存](../../_static/images/delta/delta-cache-spark-ui-storage-tab.png)

节点的磁盘使用率达到 100 %时，缓存管理器将丢弃最近使用的缓存项，以便为新数据腾出空间。

## <a name="configure-the-delta-cache"></a><a id="configure-cache"> </a><a id="configure-the-delta-cache"> </a>配置增量缓存

> [!TIP]
>
> Azure Databricks 建议为群集选择[缓存加速的工作器实例类型](#worker-instance-type)。 此类实例针对增量缓存自动执行了最佳配置。

### <a name="configure-disk-usage"></a>配置磁盘使用率

若要配置增量缓存如何使用工作器节点的本地存储，请在创建群集时指定以下 [Spark 配置](../../clusters/configure.md#spark-config)设置：

* `spark.databricks.io.cache.maxDiskUsage` - 每个节点为缓存数据预留的磁盘空间（以字节为单位）
* `spark.databricks.io.cache.maxMetaDataCache` - 每个节点为缓存元数据预留的磁盘空间（以字节为单位）
* `spark.databricks.io.cache.compression.enabled` - 是否应以压缩格式存储缓存数据

示例配置：

```ini
spark.databricks.io.cache.maxDiskUsage 50g
spark.databricks.io.cache.maxMetaDataCache 1g
spark.databricks.io.cache.compression.enabled false
```

### <a name="enable-the-delta-cache"></a><a id="enable-disable"> </a><a id="enable-the-delta-cache"> </a>启用增量缓存

若要启用和禁用增量缓存，请运行：

```scala
spark.conf.set("spark.databricks.io.cache.enabled", "[true | false]")
```

禁用缓存不会删除本地存储中已有的数据。 相反，它会阻止查询向缓存添加新数据，以及从缓存读取数据。