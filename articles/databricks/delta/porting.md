---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/05/2020
title: 迁移指南 - Azure Databricks
description: 了解如何将现有工作负载迁移到 Azure Databricks 上的 Delta Lake。
ms.openlocfilehash: 731e0ae0db930f8e4c209bb9492e38f7a6a4bad0
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121937"
---
# <a name="migration-guide"></a>迁移指南

## <a name="migrate-workloads-to-delta-lake"></a>将工作负荷迁移到 Delta Lake

在你将工作负载迁移到 Delta Lake 时，与 Apache Spark 和 Apache Hive 提供的数据源相比，你应该注意以下简化和差异。

Delta Lake 会自动处理以下操作，不应手动执行：

* `REFRESH TABLE`：Delta 表始终返回最新信息，因此无需在更改后手动调用 `REFRESH TABLE`。

* 添加和删除分区：Delta Lake 会自动跟踪表中存在的分区集，并在添加或删除数据时更新列表。  因此，无需运行 `ALTER TABLE [ADD|DROP] PARTITION` 或 `MSCK`。
* 加载单个分区：作为一种优化，你有时可以直接加载你感兴趣的数据分区。 例如，`spark.read.parquet("/data/date=2017-01-01")`。  对于 Delta Lake，这是不必要的，因为它可以从事务日志中快速读取文件列表，找到相关的文件。 如果你对单个分区感兴趣，请使用 `WHERE` 子句来指定它。 例如，`spark.read.delta("/data").where("date = '2017-01-01'")`。 对于分区中包含多个文件的大型表，此方法比从 Parquet 表加载单个分区（使用直接分区路径或 `WHERE`）的速度要快得多，因为列出目录中的文件通常比从事务日志中读取文件的列表要慢得多。

将现有应用程序移植到 Delta Lake 时，你应避免以下操作，这些操作会绕过事务日志：

* 手动修改数据：Delta Lake 使用事务日志自动提交对表的更改。 由于日志是事实的来源，因此 Spark 不会读取已写出但未添加到事务日志的文件。 同样，即使你手动删除了某个文件，指向该文件的指针仍会出现在事务日志中。 请始终使用本指南中所述的命令，而不是手动修改 Delta 表中存储的文件。

* 外部读取器：Delta Lake 中存储的数据被编码为 Parquet 文件。 但是，使用外部读取器访问这些文件并不安全。 你将看到重复和未提交的数据，并且当某人运行 [Vacuum](delta-utility.md#delta-vacuum) 时，读取可能会失败。

  > [!NOTE]
  >
  > 由于文件以开放格式编码，因此你始终可以选择将文件移到 Delta Lake 之外。  此时，可以运行 `VACUUM RETAIN 0` 并删除事务日志。 这样一来，表的文件将处于一致的状态，供你选择的外部读取器读取。

### <a name="example"></a>示例

假设你已将 Parquet 数据存储在目录 `/data-pipeline` 中，并想要创建一个名为 `events` 的表。 始终可以读入数据帧并另存为 Delta 表。 此方法将复制数据，并让 Spark 管理表。 此外，还可转换为 Delta Lake，这样速度更快，但会生成非托管表。

#### <a name="save-as-delta-table"></a>保存为 Delta 表

1. 将数据读入数据帧，并将其保存到 `delta` 格式的新目录中：

   ```python
   data = spark.read.parquet("/data-pipeline")
   data.write.format("delta").save("/mnt/delta/data-pipeline/")
   ```

2. 创建引用 Delta Lake 目录中该文件的 Delta 表 `events`：

   ```python
   spark.sql("CREATE TABLE events USING DELTA LOCATION '/mnt/delta/data-pipeline/'")
   ```

#### <a name="convert-to-delta-table"></a>转换为 Delta 表

可使用两种方法将 Parquet 表转换为 Delta 表：

* 将文件转换为 Delta Lake 格式，然后创建 Delta 表：

  ```sql
  CONVERT TO DELTA parquet.`/data-pipeline/`
  CREATE TABLE events USING DELTA LOCATION '/data-pipeline/'
  ```

* 创建 Parquet 表，然后转换为 Delta 表：

  ```sql
  CREATE TABLE events USING PARQUET OPTIONS (path '/data-pipeline/')
  CONVERT TO DELTA events
  ```

有关详细信息，请参阅[转换为 Delta](delta-utility.md#convert-to-delta)。