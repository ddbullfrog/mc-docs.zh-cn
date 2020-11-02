---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 01/08/2020
title: 创建视图 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark SQL 语言的 CREATE VIEW 语法。
ms.openlocfilehash: 8107473169fab8ce5b51dce470a28771b7d8fd9d
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473114"
---
# <a name="create-view"></a>创建视图

```sql
CREATE [OR REPLACE] [[GLOBAL] TEMPORARY] VIEW [db_name.]view_name
  [(col_name1 [COMMENT col_comment1], ...)]
  [COMMENT table_comment]
  [TBLPROPERTIES (key1=val1, key2=val2, ...)]
    AS select_statement
```

基于一个或多个表或视图定义一个逻辑视图。

**`OR REPLACE`**

如果该视图不存在，则 `CREATE OR REPLACE VIEW` 等效于 `CREATE VIEW`。 如果该视图确实存在，则 `CREATE OR REPLACE VIEW` 等效于 `ALTER VIEW`。

**`[GLOBAL] TEMPORARY`**

`TEMPORARY` 会跳过在基础元存储中持久保存视图定义（如果有）的操作。 如果指定了 `GLOBAL`，则该视图可以被不同的会话访问，并且可以在应用程序结束前保持活动状态；否则，临时视图是会话范围内的，在会话终止时会被自动删除。 所有全局临时视图都与系统保留的临时数据库 `global_temp` 相关联。 数据库名称将保留，因此不允许用户创建/使用/删除此数据库。 必须使用限定名称来访问全局临时视图。

> [!NOTE]
>
> 一个笔记本中定义的临时视图在其他笔记本中不可见。 请参阅[笔记本隔离](../../../../notebooks/notebooks-use.md#notebook-isolation)。

**`(col_name1 [COMMENT col_comment1], ...)`**

定义视图架构的列列表。 列名必须独一无二，列数必须与 `select_statement` 检索到的列数相同。 如果未指定列列表，则视图架构是 `select_statement` 的输出架构。

**`TBLPROPERTIES`**

元数据键值对。

**`AS select_statement`**

定义视图的 `SELECT` 语句。 该语句可以从基表或其他视图中进行选择。

> [!IMPORTANT]
>
> 无法指定数据源、分区或聚类分析选项，因为视图的具体化方式与表不同。

## <a name="examples"></a>示例

```sql
-- Create a persistent view view_deptDetails in database1. The view definition is recorded in the underlying metastore
CREATE VIEW database1.view_deptDetails
  AS SELECT * FROM company JOIN dept ON company.dept_id = dept.id;

-- Create or replace a local temporary view from a persistent view with an extra filter
CREATE OR REPLACE TEMPORARY VIEW temp_DeptSFO
  AS SELECT * FROM database1.view_deptDetails WHERE loc = 'SFO';

-- Access the base tables through the temporary view
SELECT * FROM temp_DeptSFO;

-- Create a global temp view to share the data through different sessions
CREATE GLOBAL TEMP VIEW global_DeptSJC
  AS SELECT * FROM database1.view_deptDetails WHERE loc = 'SJC';

-- Access the global temp views
SELECT * FROM global_temp.global_DeptSJC;

-- Drop the global temp view, temp view, and persistent view.
DROP VIEW global_temp.global_DeptSJC;
DROP VIEW temp_DeptSFO;
DROP VIEW database1.view_deptDetails;
```