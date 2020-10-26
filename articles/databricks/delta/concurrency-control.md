---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: 并发控制 - Azure Databricks
description: 了解 Delta Lake 提供的读写之间的 ACID 事务保证。
ms.openlocfilehash: a2ce38a419622c115ec484290a6e6efc3d27ba74
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121910"
---
# <a name="concurrency-control"></a>并发控制

Delta Lake 提供读写之间的 ACID 事务保证。 这表示：

* 跨多个集群的多个作者可以同时修改一个表分区，并查看表的一致性快照视图，这些写入操作将有一定的顺序。
* 读者可以继续查看 Azure Databricks 作业开始时使用的表的一致性快照视图，即使在作业过程中修改了某个表也是如此。

## <a name="optimistic-concurrency-control"></a>乐观并发控制

Delta Lake 使用[乐观并发控制](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)在写入之间提供事务保证。 在此机制下，写入操作分为三个阶段：

1. **读取** ：读取（如果需要）表的最新可用版本，以标识需要修改（即重写）的文件。
2. **写入** ：通过写入新的数据文件来暂存所有更改。
3. **验证和提交** ：在提交更改之前，检查建议的更改是否与自读取快照以来可能已同时提交的任何其他更改冲突。 如果没有冲突，则所有暂存更改都将提交为新版本的快照，并且写操作成功。 但是，如果存在冲突，写操作将失败，并出现并发修改异常，而不是像在 Parquet 表上进行写操作那样损坏表。

表的隔离级别定义必须从并发操作所作修改中隔离事务的程度。
有关 Delta Lake 在 Azure Databricks 上支持的隔离级别的详细信息，请参阅[隔离级别](optimizations/isolation-level.md)。

## <a name="write-conflicts"></a>写冲突

下表说明了在每个[隔离级别](optimizations/isolation-level.md)可能出现冲突的操作对。

|                                | INSERT                                                             | UPDATE, DELETE, MERGE INTO                         | OPTIMIZE                                           |
|--------------------------------|--------------------------------------------------------------------|----------------------------------------------------|----------------------------------------------------|
| **INSERT**                     | 不会出现冲突                                                    |                                                    |                                                    |
| **UPDATE, DELETE, MERGE INTO** | 在 Serializable 中可能出现冲突，在 WriteSerializable 中不会出现冲突 | 在 Serializable 和 WriteSerializable 中都可能出现冲突 |                                                    |
| **OPTIMIZE**                   | 不会出现冲突                                                    | 在 Serializable 和 WriteSerializable 中都可能出现冲突 | 在 Serializable 和 WriteSerializable 中都可能出现冲突 |

## <a name="avoid-conflicts-using-partitioning-and-disjoint-command-conditions"></a>使用分区和非连续命令条件来避免冲突

在所有标记为“可能出现冲突”的情况下，这两个操作是否会冲突取决于它们是否对同一组文件进行操作。 通过将表分区为与操作条件中使用的列相同的列，可以使两组文件不相交。 例如，如果未按日期对表进行分区，则两个命令 `UPDATE table WHERE date > '2010-01-01' ...` 和 `DELETE table WHERE date < '2010-01-01'` 将冲突，因为两者都可以尝试修改同一组文件。 按 `date` 对表进行分区就可以避免此冲突。 因此，根据命令上常用的条件对表进行分区可以显著减少冲突。 但是，由于存在大量子目录，因此按包含高基数的列对表进行分区可能会导致其他性能问题。

## <a name="conflict-exceptions"></a>冲突异常

发生事务冲突时，你将观察到以下异常之一：

* [ConcurrentAppendException](#concurrentappendexception)
* [ConcurrentDeleteReadException](#concurrentdeletereadexception)
* [ConcurrentDeleteDeleteException](#concurrentdeletedeleteexception)
* [MetadataChangedException](#metadatachangedexception)
* [ConcurrentTransactionException](#concurrenttransactionexception)
* [ProtocolChangedException](#protocolchangedexception)

### <a name="concurrentappendexception"></a>ConcurrentAppendException

当并发操作在操作读取的同一分区（或未分区表中的任何位置）中添加文件时，会发生此异常。 文件添加操作可能是由 `INSERT`、`DELETE`、`UPDATE` 或 `MERGE` 操作引起的。

在默认[隔离级别](optimizations/isolation-level.md)为 `WriteSerializable` 的情况下，通过盲目的 `INSERT` 操作（即，盲目追加数据而不读取任何数据的操作）添加的文件不会与任何操作冲突，即使它们接触相同的分区（或未分区表中的任何位置）也是如此。 如果隔离级别设置为 `Serializable`，则盲目追加可能会产生冲突。

通常执行 `DELETE`、`UPDATE` 或 `MERGE` 并发操作时会引发此异常。 尽管并发操作可能会物理上更新不同的分区目录，但其中一个可能会读取另一个分区同时更新的同一分区，从而导致冲突。 可以通过在操作条件中设置显式隔离来避免这种情况。 请看下面的示例。

```scala
// Target 'deltaTable' is partitioned by date and country
deltaTable.as("t").merge(
    source.as("s"),
    "s.user_id = t.user_id AND s.date = t.date AND s.country = t.country")
  .whenMatched().updateAll()
  .whenNotMatched().insertAll()
  .execute()
```

假设你在不同的日期或国家/地区同时运行上述代码。 由于每个作业都在目标 Delta 表上的独立分区上运行，因此你不会遇到任何冲突。 但是，该条件不够明确，可能会扫描整个表，并且可能与更新任何其他分区的并发操作冲突。 相反，你可以重写语句以将特定日期和国家/地区添加到合并条件中，如以下示例所示。

```scala
// Target 'deltaTable' is partitioned by date and country
deltaTable.as("t").merge(
    source.as("s"),
    "s.user_id = t.user_id AND s.date = t.date AND s.country = t.country AND t.date = '" + <date> + "' AND t.country = '" + <country> + "'")
  .whenMatched().updateAll()
  .whenNotMatched().insertAll()
  .execute()
```

现在可以安全地在不同日期和国家/地区同时运行此操作。

### <a name="concurrentdeletereadexception"></a>ConcurrentDeleteReadException

如果某个并发操作删除了你的操作读取的文件，则会发生此异常。 常见的原因是，`DELETE`、`UPDATE` 或 `MERGE` 操作导致了重写文件。

### <a name="concurrentdeletedeleteexception"></a>ConcurrentDeleteDeleteException

如果某个并发操作删除了你的操作也删除的文件，则会发生此异常。 这可能是由于两个并发 <compaction> 操作重写相同的文件引起的。

### <a name="metadatachangedexception"></a>MetadataChangedException

当并发事务更新 Delta 表的元数据时，将发生此异常。 常见原因是进行 `ALTER TABLE` 操作或写入 Delta 表以更新表的架构。

### <a name="concurrenttransactionexception"></a>ConcurrentTransactionException

如果使用同一检查点位置的流式处理查询同时启动多次，并尝试同时写入 Delta 表。 请勿让两个流式处理查询使用相同的检查点位置并同时运行。

### <a name="protocolchangedexception"></a>ProtocolChangedException

当你的 Delta 表升级到新版本时，就会发生这种情况。 为了使将来的操作成功，你可能需要升级 Delta Lake 版本。

有关详细信息，请参阅[表版本控制](versioning.md)。

<!--delta-oss-only
.. <compaction> replace:: file compaction-->

<!--delta-edge-only
.. <compaction> replace:: optimize-->