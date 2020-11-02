---
title: 事实数据表和维度表 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的事实数据表和维度表。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/23/2020
ms.date: 10/29/2020
ms.openlocfilehash: 5303c12f7549ded516bdb36d3c694d1ebe0ef40c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103559"
---
# <a name="fact-and-dimension-tables"></a>事实数据表和维度表

为 Azure 数据资源管理器数据库设计架构时，可将表宽泛地视为属于以下两种类别之一。
* 事实数据表
* 维度表

## <a name="fact-tables"></a>事实数据表
事实数据表是指其记录为不可变“事实数据”的表，例如服务日志和度量信息。 记录将以流式传输方式或大区块形式逐步追加到表中。 这些记录会一直保存在那里，直到它们因为成本考量或失去价值而被删除为止。 否则，记录永远不会更新。

实体数据有时保存在事实数据表中，这些表中的实体数据变化缓慢。 例如，有关某个物理实体（例如不经常更改位置的办公设备）的数据。
由于 Kusto 中的数据不可变，因此常见的做法是让每个表包含以下两个列：
* 用于标识实体的标识 (`string`) 列
* 上次修改时间 (`datetime`) 的时间戳列

然后，只检索每个实体标识的最后一条记录。

## <a name="dimension-tables"></a>维度表
维度表：
* 保存引用数据，例如从实体标识符到其属性在内的各种查找表
* 将类似快照的数据保存在这样的表中：表的全部内容会在单个事务中发生变化

维度表不会定期引入新数据， 而是使用 [.set-or-replace](../management/data-ingestion/ingest-from-query.md)、[.move extents](../management/move-extents.md) 或 [.rename tables](../management/rename-table-command.md) 之类的操作一次更新整个数据内容。

有时，维度表可能派生自事实数据表。 此过程可通过以下方式完成：对事实数据表执行[更新策略](../management/updatepolicy.md)，并通过对该表进行查询来获取每个实体的最后一条记录。

## <a name="differentiate-fact-and-dimension-tables"></a>区分事实数据表和维度表

Kusto 中存在区分事实数据表和维度表的进程。 其中一个是[连续导出](../management/data-export/continuous-data-export.md)。

这些机制保证一次性地精确处理事实数据表中的数据。 它们依赖于[数据库游标](../management/databasecursor.md)机制。

例如，每次执行连续导出作业都会导出自上次数据库游标更新以来引入的所有记录。 连续导出作业必须区分事实数据表和维度表。 事实数据表仅处理新引入的数据，而维度表则以查找的方式使用。 因此，必须考虑整个表。

无法将表“标记”为“事实数据表”或“维度表”。
数据引入到表中的方式以及表的使用方式决定了表的类型。
