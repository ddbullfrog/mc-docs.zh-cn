---
title: Kusto 保留策略管理 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的保留策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4896be98d5e878c8d260b1b39983931cb8a27f6a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106340"
---
# <a name="retention-policy-command"></a>保留策略命令

本文介绍用于创建和更改[保留策略](retentionpolicy.md)的控制命令。

## <a name="show-retention-policy"></a>显示保留策略

```kusto
.show <entity_type> <database_or_table_or_materialized_view> policy retention

.show <entity_type> *  policy retention
```

* `entity_type`：表、具体化视图或数据库
* `database_or_table_or_materialized_view`：`database_name` 或 `database_name.table_name` 或 `table_name`（在数据库上下文中）或 `materialized_view_name`

**示例**

显示名为 `MyDatabase` 的数据库的保留策略：

```kusto
.show database MyDatabase policy retention
```

## <a name="delete-retention-policy"></a>删除保留策略

删除数据保留策略可有效地设置无限制的数据保留。

如果删除表的数据保留策略，表将从数据库级别派生保留策略。

```kusto
.delete <entity_type> <database_or_table_or_materialized_view> policy retention
```

* `entity_type`：表、具体化视图或数据库
* `database_or_table_or_materialized_view`：`database_name` 或 `database_name.table_name` 或 `table_name`（在数据库上下文中）或 `materialized_view_name`

**示例**

删除名为 `MyTable1` 的表的保留策略：

```kusto
.delete table MyTable policy retention
```


## <a name="alter-retention-policy"></a>更改保留策略

```kusto
.alter <entity_type> <database_or_table_or_materialized_view> policy retention <retention_policy>

.alter tables (<table_name> [, ...]) policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table_or_materialized_view> policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table_or_materialized_view> policy retention [softdelete = <timespan>] [recoverability = disabled|enabled]
```

* `entity_type`：表、数据库或具体化视图
* `database_or_table_or_materialized_view`：`database_name` 或 `database_name.table_name` 或 `table_name`（在数据库上下文中）或 `materialized_view_name`
* `table_name`：数据库上下文中表的名称。  通配符（此处允许使用 `*`）。
* `retention_policy` :

```kusto
    "{ 
        \"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Disabled\"
    }" 
```

**示例**

显示名为 `MyDatabase` 的数据库的保留策略：

```kusto
.show database MyDatabase policy retention
```

设置具有 10 天软删除期且禁用数据可恢复性的保留策略：

```kusto
.alter-merge table Table1 policy retention softdelete = 10d recoverability = disabled

.alter-merge materialized-view View1 policy retention softdelete = 10d recoverability = disabled
```

设置具有 10 天软删除期且启用数据可恢复性的保留策略：

```kusto
.alter table Table1 policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"

.alter materialized-view View1 policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

设置与上述相同的保留策略，但这一次适用于多个表（Table1、Table2 和 Table3）：

```kusto
.alter tables (Table1, Table2, Table3) policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

设置使用默认值的保留策略：软删除期为 100 年并启用可恢复性：

```kusto
.alter table Table1 policy retention "{}"
```
