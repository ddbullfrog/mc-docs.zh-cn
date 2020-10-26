---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: SQL 参考 - Azure Databricks
description: 了解 Azure Databricks 中支持的 Apache Spark 和 Delta Lake SQL 语言构造及其示例用例。
keywords: Spark SQL, 参考
ms.openlocfilehash: c91998a43a584a2009f48bed3a63c7ecc23f97e2
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473018"
---
# <a name="sql-reference"></a>SQL 参考

下面是在 Azure Databricks 中受支持且适用于Apache Spark SQL 和 Delta Lake 的数据定义语言 (DDL) 和数据操作语言 (DML) 构造的完整列表。

> [!NOTE]
>
> 如果要编写 SQL 查询，无论是在 SQL 笔记本中编写还是使用不同的默认语言在笔记本的 `%sql` [magic 命令](../../../../notebooks/notebooks-use.md#language-magic)中编写，均不能在标识符中使用 `$`，因为它会被解释为参数。 若要在 SQL 命令单元中转义 `$`，请使用 `$\`。 例如，若要定义标识符 `$foo`，请将其编写为 `$\foo`。

* [更改数据库](alter-database.md)
* [更改表或视图](alter-table-or-view.md)
* [更改表分区](alter-table-partition.md)
* [分析表](analyze-table.md)
* [缓存](cache-dbio.md)
* [缓存表](cache-table.md)
* [清除缓存](clear-cache.md)
* [克隆（Azure Databricks 上的 Delta Lake）](clone.md)
* [转换为 Delta（Azure Databricks 上的 Delta Lake）](convert-to-delta.md)
* [复制到（Azure Databricks 上的 Delta Lake）](copy-into.md)
* [创建 Bloom 筛选器索引](create-bloomfilter-index.md)
* [创建数据库](create-database.md)
* [创建函数](create-function.md)
* [创建表](create-table.md)
* [创建视图](create-view.md)
* [删除（Azure Databricks 上的 Delta Lake）](delete-from.md)
* [拒绝](deny.md)
* [描述数据库](describe-database.md)
* [描述函数](describe-function.md)
* [描述表](describe-table.md)
* [描述历史记录（Azure Databricks 上的 Delta Lake）](describe-history.md)
* [删除 Bloom 筛选器索引](drop-bloomfilter-index.md)
* [删除数据库](drop-database.md)
* [删除函数](drop-function.md)
* [删除表](drop-table.md)
* [删除视图](drop-view.md)
* [解释](explain.md)
* [对表进行 Fsck 修复（Azure Databricks 上的 Delta Lake）](fsck.md)
* [函数](functions.md)
* [授予](grant.md)
* [插入](insert.md)
* [加载数据](load-data.md)
* [合并到（Azure Databricks 上的 Delta Lake）](merge-into.md)
* [Msck](msck.md)
* [优化（Azure Databricks 上的 Delta Lake）](optimize.md)
* [刷新表](refresh-table.md)
* [重置](reset.md)
* [撤销](revoke.md)
* [Select](select.md)
* [设置](set.md)
* [显示列](show-columns.md)
* [显示用于创建表的命令](show-create-table.md)
* [显示数据库](show-databases.md)
* [显示函数](show-functions.md)
* [显示授权](show-grant.md)
* [显示分区](show-partitions.md)
* [显示表属性](show-table-properties.md)
* [显示表](show-tables.md)
* [截断表](truncate-table.md)
* [取消缓存表](uncache-table.md)
* [更新（Azure Databricks 上的 Delta Lake）](update.md)
* [使用数据库](use-database.md)
* [清空](vacuum.md)