---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/13/2020
title: 更改表或视图 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用 Apache Spark 和 Delta Lake SQL 语言的 ALTER TABLE 和 ALTER VIEW 语法。
ms.openlocfilehash: 7ce650dc617a306086714c1dfda01942bcb9a29b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472753"
---
# <a name="alter-table-or-view"></a>更改表或视图

## <a name="rename-table-or-view"></a>重命名表或视图

```sql
ALTER [TABLE|VIEW] [db_name.]table_name RENAME TO [db_name.]new_table_name
```

重命名一个现有的表或视图。 如果目标表名称已存在，则会引发异常。 此操作不支持跨数据库移动表。

对于托管表，重命名表将移动表位置；对于非托管（外部）表，重命名表不会移动表位置。

有关托管和非托管（外部）表的详细信息，请参阅[托管和非托管表](../../../../data/tables.md#managed-unmanaged-tables)。

## <a name="set-table-or-view-properties"></a>设置表或视图属性

```sql
ALTER [TABLE|VIEW] table_name SET TBLPROPERTIES (key1=val1, key2=val2, ...)
```

设置现有表或视图的属性。 如果已设置特定属性，这将用新属性覆盖旧值。

> [!NOTE]
>
> * 属性名称区分大小写。 如果有 `key1`，并在稍后设置 `Key1`，则将创建新的表属性。
> * 若要查看表属性，请运行：
>
>   ```sql
>   DESCRIBE EXTENDED table_name
>   ```

### <a name="set-a-table-comment"></a>设置表注释

若要设置表注释，请运行：

```sql
ALTER TABLE table_name SET TBLPROPERTIES ('comment' = 'A table comment.')
```

## <a name="drop-table-or-view-properties"></a>删除表或视图属性

```sql
ALTER (TABLE|VIEW) table_name UNSET TBLPROPERTIES
    [IF EXISTS] (key1, key2, ...)
```

删除现有表或视图的一个或多个属性。 如果指定的属性不存在，会引发异常。

**`IF EXISTS`**

如果指定的属性不存在，将不会发生任何情况。

## <a name="set-serde-or-serde-properties"></a>设置 SerDe 或 SerDe 属性

```sql
ALTER TABLE table_name [PARTITION part_spec] SET SERDE serde
    [WITH SERDEPROPERTIES (key1=val1, key2=val2, ...)]

ALTER TABLE table_name [PARTITION part_spec]
    SET SERDEPROPERTIES (key1=val1, key2=val2, ...)

part_spec:
    : (part_col_name1=val1, part_col_name2=val2, ...)
```

设置表或分区的 SerDe 或 SerDe 属性。 如果已设置指定的 SerDe 属性，这将用新属性覆盖旧值。 仅允许使用 Hive 格式创建的表设置 SerDe。

## <a name="assign-owner"></a>分配所有者

```sql
ALTER (TABLE|VIEW) object-name OWNER TO `user_name@user_domain.com`
```

为表或视图分配所有者。

## <a name="delta-lake-schema-constructs"></a>Delta Lake 架构构造

Delta Lake 还支持其他可修改表架构的构造：添加、更改和替换列。

有关添加、更改和替换列示例的详细说明，请参阅[显式更新架构](../../../../delta/delta-batch.md#explicit-schema-update)。

### <a name="add-columns"></a>添加列

```sql
ALTER TABLE table_name ADD COLUMNS (col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name], ...)

ALTER TABLE table_name ADD COLUMNS (col_name.nested_col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name], ...)
```

向现有表添加列。 它支持添加嵌套列。 如果表中已存在具有相同名称的列或同一嵌套结构，会引发异常。

### <a name="change-columns"></a>更改列

```sql
ALTER TABLE table_name (ALTER|CHANGE) [COLUMN] alterColumnAction

ALTER TABLE table_name (ALTER|CHANGE) [COLUMN] alterColumnAction

alterColumnAction:
    : TYPE dataType
    : [COMMENT col_comment]
    : [FIRST|AFTER colA_name]
    : (SET | DROP) NOT NULL
```

更改现有表的列定义。 可以更改数据类型、注释、列的为空性或对列重新排序。

> [!NOTE]
>
> 在 Databricks Runtime 7.0 及更高版本中可用。

### <a name="change-columns-hive-syntax"></a>更改列（Hive 语法）

```sql
ALTER TABLE table_name CHANGE [COLUMN] col_name col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name]

ALTER TABLE table_name CHANGE [COLUMN] col_name.nested_col_name col_name data_type [COMMENT col_comment] [FIRST|AFTER colA_name]
```

更改现有表的列定义。 可以更改列的注释并对列重新排序。

> [!NOTE]
>
> 在 Databricks Runtime 7.0 及更高版本中你无法使用 `CHANGE COLUMN`：
>
> * 要更改复杂数据类型（如结构）的内容。 改用 `ADD COLUMNS` 来将新列添加到嵌套字段，或使用 `ALTER COLUMN` 更改嵌套列的属性。
> * 若要放宽 Delta 表中列的为空性。 改用 `ALTER TABLE table_name ALTER COLUMN column_name DROP NOT NULL`。

### <a name="replace-columns"></a>替换列

```sql
ALTER TABLE table_name REPLACE COLUMNS (col_name1 col_type1 [COMMENT col_comment1], ...)
```

替换现有表的列定义。 它支持更改列的注释、添加列和对列重新排序。 如果指定的列定义与现有定义不兼容，则引发异常。