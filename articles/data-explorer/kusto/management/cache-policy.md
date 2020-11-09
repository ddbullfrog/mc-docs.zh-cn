---
title: 缓存策略 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的缓存策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4bac704d4915f14c85631fb4dcd0f309b62f67a9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105224"
---
# <a name="cache-policy-command"></a>缓存策略命令

本文介绍用于创建和更改[缓存策略](cachepolicy.md)的命令 

## <a name="displaying-the-cache-policy"></a>显示缓存策略

可以对数据库、表或[具体化视图](materialized-views/materialized-view-overview.md)设置此策略，并可以使用以下命令之一来显示此策略：

* `.show` `database` DatabaseName `policy` `caching`
* `.show` `table` TableName `policy` `caching`
* `.show` `materialized-view` *MaterializedViewName* `policy` `caching`

## <a name="altering-the-cache-policy"></a>更改缓存策略

```kusto
.alter <entity_type> <database_or_table_or_materialized-view_name> policy caching hot = <timespan>
```

更改多个表的缓存策略（在相同数据库上下文中）：

```kusto
.alter tables (table_name [, ...]) policy caching hot = <timespan>
```

缓存策略：

```kusto
{
  "DataHotSpan": {
    "Value": "3.00:00:00"
  },
  "IndexHotSpan": {
    "Value": "3.00:00:00"
  }
}
```

* `entity_type`：表、具体化视图、数据库或群集
* `database_or_table_or_materialized-view`：如果实体为表或数据库，则应在命令中指定其名称，如下所示： 
  - `database_name` 或 
  - `database_name.table_name` 或 
  - `table_name`（在特定数据库的上下文中运行时）

## <a name="deleting-the-cache-policy"></a>删除缓存策略

```kusto
.delete <entity_type> <database_or_table_name> policy caching
```

**示例**

显示数据库 `MyDatabase` 中表 `MyTable` 的缓存策略：

```kusto
.show table MyDatabase.MyTable policy caching 
```

将表 `MyTable` 的缓存策略（在数据库上下文中）设置为 3 天：

```kusto
.alter table MyTable policy caching hot = 3d
.alter materialized-view MyMaterializedView policy caching hot = 3d
```

将多个表的策略（在数据库上下文中）设置为 3 天：

```kusto
.alter tables (MyTable1, MyTable2, MyTable3) policy caching hot = 3d
```

删除针对表设置的策略：

```kusto
.delete table MyTable policy caching
```

删除针对具体化视图设置的策略：

```kusto
.delete materialized-view MyMaterializedView policy caching
```

删除针对数据库设置的策略：

```kusto
.delete database MyDatabase policy caching
```
