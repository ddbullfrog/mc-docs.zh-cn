---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 隔离级别 - Azure Databricks
description: 了解 Azure Databricks 上的 Delta Lake 在 Delta 表上执行并发事务时提供的隔离级别。
ms.openlocfilehash: 721ef5fcc8a263f9f6ae64e838ffcbc0b25de620
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121939"
---
# <a name="isolation-levels"></a>隔离级别

表的隔离级别定义了必须将某事务与并发事务所作修改进行隔离的程度。 Azure Databricks 上的 Delta Lake 支持两个隔离级别：Serializable 和 WriteSerializable。

* **Serializable** ：最强隔离级别。 它可确保提交的写入操作和所有读取均[可序列化](https://en.wikipedia.org/wiki/Serializability)。 只要有一个串行序列一次执行一项操作，且生成与表中所示相同的结果，则可执行这些操作。 对于写入操作，串行序列与表的历史记录中所示完全相同。
* **WriteSerializable（默认）** ：强度比 Serializable 低的隔离级别。 它仅确保写入操作（而非读取）可序列化。 但是，这仍比[快照](https://en.wikipedia.org/wiki/Snapshot_isolation)隔离更安全。 WriteSerializable 是默认的隔离级别，因为对大多数常见操作而言，它使数据一致性和可用性之间达到良好的平衡。

  在此模式下，Delta 表的内容可能与[表历史记录](../../spark/latest/spark-sql/language-manual/describe-table.md)中所示的操作序列不同。 这是因为此模式允许某些并发写入对（例如操作 X 和 Y）继续执行，这样的话，即使历史记录显示在 X 之后提交了 Y，结果也像在 X 之前执行 Y 一样（即它们之间是可序列化的）。若要禁止这种重新排序，请将[表隔离级别设置](#setting-isolation-level)为 Serializable，以使这些事务失败。

若要详细了解哪些类型的操作可能会在每个隔离级别彼此冲突以及可能出现的错误，请参阅[并发控制](../concurrency-control.md)。

## <a name="set-the-isolation-level"></a><a id="set-the-isolation-level"> </a><a id="setting-isolation-level"> </a>设置隔离级别

使用 `ALTER TABLE` 命令设置隔离级别。

```sql
ALTER TABLE <table-name> SET TBLPROPERTIES ('delta.isolationLevel' = <level-name>)
```

其中，`<level-name>` 是 `Serializable` 或 `WriteSerializable`。

例如，若要将隔离级别从默认的 `WriteSerializable` 更改为 `Serializable`，请运行：

```sql
ALTER TABLE <table-name> SET TBLPROPERTIES ('delta.isolationLevel' = 'Serializable')
```