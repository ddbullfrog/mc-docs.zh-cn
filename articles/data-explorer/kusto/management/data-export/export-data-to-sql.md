---
title: 将数据导出到 SQL - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了如何在 Azure 数据资源管理器中将数据导出到 SQL。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: ce52ed9f0b53ae80c585f7a3ad6786724d0f5699
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106271"
---
# <a name="export-data-to-sql"></a>将数据导出到 SQL

“将数据导出到 SQL”可以运行查询并将其结果发送到 SQL 数据库（例如由 Azure SQL 数据库服务承载的 SQL 数据库）中的表。

**语法**

`.export` [`async`] `to` `sql` *SqlTableName* *SqlConnectionString* [`with` `(`*PropertyName* `=` *PropertyValue*`,`...`)`] `<|` *Query*

其中：
* async：命令以异步模式运行（可选）。
* SqlTableName：要在其中插入数据的 SQL 数据库表的名称。
  为了防范注入式攻击，此名称具有限制。
* SqlConnectionString 是遵循 `ADO.NET` 连接字符串格式的 `string` 文本，用于描述要连接到的 SQL 终结点和数据库。 出于安全原因，连接字符串具有限制。
* PropertyName 和 PropertyValue 是配对的名称（标识符）和值（字符串文本）。

属性：

|名称               |值           |说明|
|-------------------|-----------------|-----------|
|`firetriggers`     |`true` 或 `false`|如果为 `true`，则指示目标系统激发 SQL 表上定义的 INSERT 触发器。 默认为 `false`。 （有关详细信息，请参阅 [BULK INSERT](https://docs.microsoft.com/sql/t-sql/statements/bulk-insert-transact-sql) 和 [System.Data.SqlClient.SqlBulkCopy](https://docs.microsoft.com/dotnet/api/system.data.sqlclient.sqlbulkcopy)）|
|`createifnotexists`|`true` 或 `false`|如果为 `true`，则会在目标 SQL 表不存在的情况下创建该表；在这种情况下，必须提供 `primarykey` 属性来指示作为主键的结果列。 默认为 `false`。|
|`primarykey`       |                 |如果 `createifnotexists` 为 `true`，则指示结果中将用作 SQL 表主键的列的名称（如果该表是由此命令创建的）。|
|`persistDetails`   |`bool`           |指示此命令应保留其结果（请参阅 `async` 标志）。 在异步运行中默认为 `true`，但如果调用方不需要结果，则可将其关闭。 在同步执行中默认为 `false`，但可以开启。 |
|`token`            |`string`         |AAD 访问令牌，Kusto 会将其转发到用于身份验证的 SQL 终结点。 设置后，SQL 连接字符串不应包含身份验证信息，如 `Authentication`、`User ID` 或 `Password`。|

## <a name="limitations-and-restrictions"></a>限制和局限

将数据导出到 SQL 数据库时，有许多限制和局限：

1. Kusto 是一种云服务，因此，连接字符串必须指向可从云中访问的数据库。 （具体说来，无法导出到本地数据库，因为无法从公有云访问它。）

2. 当调用方主体为 Azure Active Directory 主体（`aaduser=` 或 `aadapp=`）时，Kusto 支持 Active Directory 集成身份验证。
   另外，Kusto 还支持在连接字符串中提供 SQL 数据库的凭据。 不支持其他身份验证方法。 提供给 SQL 数据库的标识始终来自命令调用方，而不是来自 Kusto 服务标识本身。

3. 如果 SQL 数据库中的目标表存在，则它必须与查询结果架构匹配。 请注意，在某些情况下（例如，数据库为 Azure SQL 数据库），这意味着表中有一个列标记为标识列。

4. 导出大量数据可能需要很长时间。 批量导入期间，建议在设置目标 SQL 表时尽量减少日志记录。
   请参阅 [SQL Server 数据库引擎 > ... > 数据库功能 > 批量导入和导出数据](https://docs.microsoft.com/sql/relational-databases/import-export/prerequisites-for-minimal-logging-in-bulk-import)。

5. 数据导出是使用 SQL 大容量复制执行的，在目标 SQL 数据库上不提供事务性保证。 如需更多详细信息，请参阅[事务和大容量复制操作](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/transaction-and-bulk-copy-operations)。

6. SQL 表名称被限制为包含字母、数字、空格、下划线 (`_`)、点 (`.`) 和连字符 (`-`) 的名称。

7. SQL 连接字符串限制如下：`Persist Security Info` 显式设置为 `false`，`Encrypt` 设置为 `true`，`Trust Server Certificate` 设置为 `false`。

8. 创建新的 SQL 表时，可以指定列的主键属性。 如果列的类型为 `string`，则 SQL 可能会因对主键列的其他限制而拒绝创建表。 解决方法是在导出数据前手动在 SQL 中创建表。 存在此限制的原因在于，SQL 中的主键列不能是无限大小，但 Kusto 表列没有已声明的大小限制。

## <a name="azure-db-aad-integrated-authentication-documentation"></a>Azure DB AAD 集成身份验证文档

* [使用 Azure Active Directory 身份验证进行 SQL 数据库身份验证](/sql-database/sql-database-aad-authentication)
* [适用于 Azure SQL DB 和 SQL DW 工具的 Azure AD 身份验证扩展] (https://azure.microsoft.com/blog/azure-ad-authentication-extensions-for-azure-sql-db-and-sql-dw-tools/)

**示例** 

在此示例中，Kusto 运行查询，然后将查询生成的第一个记录集导出到服务器 `myserver` 的 `MyDatabase` 数据库中的 `MySqlTable` 表。

```kusto 
.export async to sql MySqlTable
    h@"Server=tcp:myserver.database.chinacloudapi.cn,1433;Database=MyDatabase;Authentication=Active Directory Integrated;Connection Timeout=30;"
    <| print Id="d3b68d12-cbd3-428b-807f-2c740f561989", Name="YSO4", DateOfBirth=datetime(2017-10-15)
```

在此示例中，Kusto 运行查询，然后将查询生成的第一个记录集导出到服务器 `myserver` 的 `MyDatabase` 数据库中的 `MySqlTable` 表。
如果目标表在目标数据库中不存在，则创建它。

```kusto 
.export async to sql ['dbo.MySqlTable']
    h@"Server=tcp:myserver.database.chinacloudapi.cn,1433;Database=MyDatabase;Authentication=Active Directory Integrated;Connection Timeout=30;"
    with (createifnotexists="true", primarykey="Id")
    <| print Message = "Hello World!", Timestamp = now(), Id=12345678
```