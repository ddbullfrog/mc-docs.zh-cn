---
title: 外部表 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的外部表。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 01/22/2020
ms.date: 10/29/2020
ms.openlocfilehash: c4c902d79eea123420cbc4342ca039c2af149fa5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106199"
---
# <a name="external-tables"></a>外部表

外部表是引用存储在 Azure 数据资源管理器数据库外部的数据的 Kusto 架构实体。

与[表](tables.md)类似，外部表具有明确定义的架构（列名称和数据类型对的已排序列表）。 与表不同，数据是在群集外部进行存储和管理的。 大多数情况下，数据以某些标准格式（例如 CSV、Parquet、Avro）存储，而不是由 Azure 数据资源管理器引入。

外部表只创建一次。 请参阅以下命令创建外部表：
* [外部表常规控制命令](../../management/external-table-commands.md)
* [创建和更改外部 SQL 表](../../management/external-sql-tables.md)
* [在 Azure 存储或 Azure Data Lake 中创建和更改表](../../management/external-tables-azurestorage-azuredatalake.md)

可使用 [external_table()](../../query/externaltablefunction.md) 函数按外部表名称引用外部表。 

**说明**

* 外部表名称：
   * 区分大小写。
   * 不能与 Kusto 表名重叠。
   * 需遵循[实体名称](./entity-names.md)的规则。
* 每个数据库的外部表数量上限为 1,000。
* Kusto 支持[导出](../../management/data-export/export-data-to-an-external-table.md)和[连续导出](../../management/data-export/continuous-data-export.md)到外部表，以及[查询外部表](../../../data-lake-query-data.md)。
* [数据清除](../../concepts/data-purge.md)未应用于外部表。 记录永远不会从外部表中删除。
