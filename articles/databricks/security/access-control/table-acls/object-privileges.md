---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: 数据对象权限 - Azure Databricks
description: 了解如何在 Azure Databricks 中设置表、数据库、视图、函数及其子集的权限。
ms.openlocfilehash: 098f9aa5a342682665311254b6ded1059a0b7422
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937747"
---
# <a name="data-object-privileges"></a>数据对象特权

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

利用 Azure Databricks 数据治理模型，你可以以编程方式授予、拒绝和撤消对来自 Spark SQL API 数据的访问权限。 利用此模型，你可以控制对安全对象（如表、数据库、视图和函数）的访问权限。 它还允许对从任意查询创建的派生视图设置权限，从而进行精细的访问控制（例如对表的特定子集）。 Azure Databricks SQL 查询分析器在启用了表访问控制的群集上，在运行时强制执行这些访问控制策略。

本文将介绍组成 Azure Databricks 数据治理模型的权限、对象和所有权规则。 还将介绍如何授予、拒绝和撤消对象权限。

## <a name="requirements"></a>要求

在管理员或对象所有者可以对数据对象授予、拒绝或撤消权限之前：

* 管理员必须为工作区[启用并强制执行表访问控制](../../../administration-guide/access-control/table-acl.md)。
* 必须启用群集才能进行[表访问控制](table-acl.md#table-access-control)。

## <a name="data-governance-model"></a>数据治理模型

本部分介绍 Azure Databricks 的数据治理模型。 对数据对象的访问权限由权限控制。

### <a name="privileges"></a>权限

* `SELECT` - 为对象提供读取访问权限
* `CREATE` - 提供创建对象（例如数据库中的表）的功能
* `MODIFY` - 提供在对象中添加、删除和修改数据的功能
* `READ_METADATA` - 提供查看对象及其元数据的功能
* `CREATE_NAMED_FUNCTION` - 提供在现有的目录或数据库中创建命名 UDF 的功能
* `ALL PRIVILEGES` - 提供所有权限（转换为上述所有权限）

#### <a name="privilege-hierarchy"></a>权限层次结构

对象上的权限是分层的。 这意味着向 `CATALOG` 对象授予或拒绝某项权限会自动向其所有包含的数据库（以及所有表和视图）授予或拒绝该权限。 同样，向 `DATABASE` 授予或拒绝某项权限将自动向该数据库中所有表和视图授予或拒绝该权限。

### <a name="objects"></a>对象

权限可应用于下列对象的类：

* `CATALOG` - 控制对整个数据目录的访问。
* `DATABASE` - 控制对数据库的访问。
* `TABLE` - 控制对托管表或外部表的访问。
* `VIEW` - 控制对 SQL 视图的访问。
* `FUNCTION` - 控制对命名函数的访问。
* `ANONYMOUS FUNCTION` - 控制对[匿名函数或临时函数](../../../spark/latest/spark-sql/language-manual/create-function.md)的访问。
* `ANY FILE` - 控制对基础文件系统的访问。

### <a name="object-ownership"></a>对象所有权

在群集上启用表 ACL 后，创建数据库、表、视图或函数的用户将成为其所有者。 所有者被授予所有权限，并且可以向其他用户授予权限。

所有权决定是否可以将派生对象的权限授予其他用户。
例如，假设用户 A 拥有表 T，并授予用户 B 对表 T 的 `SELECT` 权限。即使用户 B 可以从表 T 中进行选择，用户 B 也不能向用户 C 授予对表 T 的 `SELECT` 权限，因为用户 A 仍是基础表 T 的所有者。此外，用户 B 不能仅通过在表 T 上创建视图 V，并向用户 C 授予对该视图的权限来规避此限制。当 Azure Databricks 检查用户 C 访问视图 V 的权限时，还会检查 V 和基础表 T 的所有者是否相同。 如果所有者不相同，则用户 C 还必须对基础表 T 具有 `SELECT` 权限。

在群集上禁用表 ACL 时，创建数据库、表、视图或函数时不会注册所有者。 若要测试某个对象是否具有所有者，请运行 `SHOW GRANT ON <object-name>`。
如果未看到带有 `ActionType OWN` 的条目，则该对象没有所有者。

#### <a name="assign-owner-to-object"></a>为对象分配所有者

管理员可以使用 ` ALTER <object> OWNER TO `<user-name>@<user-domain>.com` ` 命令为对象分配所有者：

```sql
ALTER DATABASE <database-name> OWNER TO `<user-name>@<user-domain>.com`
ALTER TABLE <table-name> OWNER TO `<user-name>@<user-domain>.com`
ALTER VIEW <view-name> OWNER TO `<user-name>@<user-domain>.com`
```

### <a name="users-and-groups"></a>用户和组

管理员和所有者可以向使用[组 API](../../../dev-tools/api/latest/groups.md) 创建的用户和组授予权限。
每个用户都通过其在 Azure Databricks 中的用户名中唯一标识（通常映射到其电子邮件地址）。

> [!NOTE]
>
> 必须将用户规范括在反引号 (``) 中，而不是单引号 (‘’) 中。

### <a name="operations-and-privileges"></a>操作和权限

下表将 SQL 操作映射到执行该操作所需的权限或角色：

| 操作/权限或角色                                                           | `SELECT`   | `CREATE`   | `MODIFY`   | `READ_METADATA`   | `CREATE_NAMED_FUNCTION`   | 所有者   | 管理员   |
|-----------------------------------------------------------------------------------------|------------|------------|------------|-------------------|---------------------------|---------|---------|
| [CREATE DATABASE](../../../spark/latest/spark-sql/language-manual/create-database.md)   |            | x          |            |                   |                           | x       | x       |
| [CREATE TABLE](../../../spark/latest/spark-sql/language-manual/create-table.md)         |            | x          |            |                   |                           | x       | x       |
| [CREATE VIEW](../../../spark/latest/spark-sql/language-manual/create-view.md)           |            | x          |            |                   |                           | x       | x       |
| [CREATE FUNCTION](../../../spark/latest/spark-sql/language-manual/create-function.md)   |            |            |            |                   | x                         | x       | x       |
| [ALTER DATABASE](../../../spark/latest/spark-sql/language-manual/alter-database.md)     |            |            |            |                   |                           | x       | x       |
| [ALTER TABLE](../../../spark/latest/spark-sql/language-manual/alter-table-or-view.md)   |            |            |            |                   |                           | x       | x       |
| [ALTER VIEW](../../../spark/latest/spark-sql/language-manual/alter-table-or-view.md)    |            |            |            |                   |                           | x       | x       |
| [DROP DATABASE](../../../spark/latest/spark-sql/language-manual/drop-database.md)       |            |            |            |                   |                           | x       | x       |
| [DROP TABLE](../../../spark/latest/spark-sql/language-manual/drop-table.md)             |            |            |            |                   |                           | x       | x       |
| [DROP VIEW](../../../spark/latest/spark-sql/language-manual/drop-view.md)               |            |            |            |                   |                           | x       | x       |
| [.DROP FUNCTION](../../../spark/latest/spark-sql/language-manual/drop-function.md)       |            |            |            |                   |                           | x       | x       |
| [DESCRIBE TABLE](../../../spark/latest/spark-sql/language-manual/describe-table.md)     |            |            |            | x                 |                           | x       | x       |
| [EXPLAIN](../../../spark/latest/spark-sql/language-manual/explain.md)                   |            |            |            | x                 |                           | x       | x       |
| [DESCRIBE HISTORY](../../../spark/latest/spark-sql/language-manual/describe-history.md) |            |            |            |                   |                           | x       | x       |
| [SELECT](../../../spark/latest/spark-sql/language-manual/select.md)                     | x          |            |            |                   |                           | x       | x       |
| [INSERT](../../../spark/latest/spark-sql/language-manual/insert.md)                     |            |            | x          |                   |                           | x       | x       |
| [UPDATE](../../../spark/latest/spark-sql/language-manual/update.md)                     |            |            | x          |                   |                           | x       | x       |
| [MERGE INTO](../../../spark/latest/spark-sql/language-manual/merge-into.md)             |            |            | x          |                   |                           | x       | x       |
| [DELETE FROM](../../../spark/latest/spark-sql/language-manual/delete-from.md)           |            |            | x          |                   |                           | x       | x       |
| [TRUNCATE TABLE](../../../spark/latest/spark-sql/language-manual/truncate-table.md)     |            |            | x          |                   |                           | x       | x       |
| [OPTIMIZE](../../../spark/latest/spark-sql/language-manual/optimize.md)                 |            |            | x          |                   |                           | x       | x       |
| [VACUUM](../../../spark/latest/spark-sql/language-manual/vacuum.md)                     |            |            | x          |                   |                           | x       | x       |
| [FSCK REPAIR TABLE](../../../spark/latest/spark-sql/language-manual/fsck.md)            |            |            | x          |                   |                           | x       | x       |
| [MSCK](../../../spark/latest/spark-sql/language-manual/msck.md)                         |            |            |            |                   |                           | x       | x       |
| [GRANT](../../../spark/latest/spark-sql/language-manual/grant.md)                       |            |            |            |                   |                           | x       | x       |
| [DENY](../../../spark/latest/spark-sql/language-manual/deny.md)                         |            |            |            |                   |                           | x       | x       |
| [REVOKE](../../../spark/latest/spark-sql/language-manual/revoke.md)                     |            |            |            |                   |                           | x       | x       |

> [!IMPORTANT]
>
> 使用表 ACL 时，`DROP TABLE` 语句区分大小写。 如果表名称为小写，且 drop table 使用混合或大写形式引用表名称，则 `DROP TABLE` 语句将失败。

## <a name="manage-object-privileges"></a>管理对象权限

可以使用 `GRANT`、`DENY` 和 `REVOKE` 操作来管理对象权限。

> [!NOTE]
>
> * 对象的所有者或管理员可以执行 `GRANT`、`DENY` 和 `REVOKE` 操作。 但是，管理员不能拒绝所有者的权限或撤消所有者的权限。
> * 不是所有者或管理员的主体只能在授予了所需的权限后才可以执行操作。
> * 若要为所有用户授予、拒绝或撤消权限，请在 `TO` 之后指定关键字 `users`。

**示例**

```sql
GRANT SELECT ON DATABASE <database-name> TO `<user>@<domain-name>`
GRANT SELECT ON ANONYMOUS FUNCTION TO `<user>@<domain-name>`
GRANT SELECT ON ANY FILE TO `<user>@<domain-name>`

SHOW GRANT `<user>@<domain-name>` ON DATABASE <database-name>

DENY SELECT ON <table-name> TO `<user>@<domain-name>`

REVOKE ALL PRIVILEGES ON DATABASE default FROM `<user>@<domain-name>`
REVOKE SELECT ON <table-name> FROM `<user>@<domain-name>`

GRANT SELECT ON ANY FILE TO users
```

## <a name="frequently-asked-questions-faq"></a>常见问题解答 (FAQ)

**如何为所有用户授予、拒绝或撤消权限**

在 `TO` 或 `FROM` 之后指定关键字 `users`。 例如：

```sql
GRANT SELECT ON ANY FILE TO users
```

**我创建了一个对象，但现在不能对其进行查询、删除或修改。**

出现此错误的原因可能是你在未启用表 ACL 的群集上创建了该对象。 在群集上禁用表 ACL 时，创建数据库、表或视图时不会注册所有者。 管理员必须使用以下命令为对象分配所有者：

```sql
ALTER [DATABASE|TABLE|VIEW] <object-name> OWNER TO `<user-name>@<user-domain>.com`;
```

**如何授予全局和本地临时视图的权限？**

很遗憾，不支持全局和本地临时视图的权限。 本地临时视图仅在同一会话中可见，在 `global_temp` 数据库中创建的视图对共享群集的所有用户可见。 但是，会强制执行对任何临时视图所引用的基础表和视图的权限。

**如何同时向用户或组授予多个表的权限？**

授予、拒绝或撤销语句一次只能应用于一个对象。 建议通过数据库向主体组织并授予多个表的权限。 如果授予数据库的主体 `SELECT` 权限，则会向该数据库中的所有表和视图隐式授予该主体 `SELECT` 权限。 例如，如果数据库 D 具有表 t1 和 t2，并且管理员发出以下 `GRANT` 命令：

```sql
GRANT SELECT ON DATABASE D TO `<user>@<domain-name>`
```

主体 `<user>@<domain-name>` 可以从表 t1 和 t2 中进行选择，还可以从将来在数据库 D 中创建的任何表和视图进行选择。

**如何向用户授予除一个表之外的所有表的权限？**

向数据库授予 `SELECT` 权限，然后拒绝要限制访问的特定表的 `SELECT` 权限。

```sql
GRANT SELECT ON DATABASE D TO `<user>@<domain-name>`
DENY SELECT ON TABLE D.T TO `<user>@<domain-name>`
```

主体 `<user>@<domain-name>` 可以从 D 中的所有表中选择（D.T 除外）。

**用户对表 T 的视图具有 `SELECT` 权限，但当该用户尝试从该视图中 `SELECT` 时，他们会收到错误 `User does not have privilege SELECT on table`。**

此常见错误可能是由以下任一原因所致：

* 表 T 没有注册所有者，因为它是使用禁用了表 ACL 的群集创建的。
* 表 T 视图上的 `SELECT` 权限的授予者不是表 T 的所有者，或者用户在表 T 上也没有 `SELECT` 权限。

假设有一个表 T 由 A 拥有。A 拥有 T 上的视图 V1，B 拥有 T 上的视图 V2。

* 如果 A 已授予对视图 V1 的 `SELECT` 权限，则用户可以在 V1 上选择。
* 如果 A 已授予对表 T 的 `SELECT` 权限，并且 B 已授予对 V2 的 `SELECT` 权限，则用户可以在 V2 上选择。

如[对象所有权](#object-ownership)部分中所述，这些条件确保只有对象的所有者才能向其他用户授予对该对象的访问权限。

**我尝试在已启用表 ACL 的群集上运行 `sc.parallelize`，但失败了。**

在已启用表 ACL 的群集上，只能使用 Spark SQL 和 Python 数据帧 API。 出于安全原因，不允许使用 RDD API，因为 Azure Databricks 无法检查和授权 RDD 中的代码。