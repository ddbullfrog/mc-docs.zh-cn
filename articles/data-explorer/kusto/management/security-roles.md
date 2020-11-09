---
title: 安全角色管理 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的安全角色管理。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 83eb67db1df3c1a6eb69ea85a4ad9625b2f824c7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106313"
---
# <a name="security-roles-management"></a>安全角色管理

> [!IMPORTANT]
> 在 Kusto 群集上更改授权规则之前，请阅读以下内容：[Kusto 访问控制概述](../management/access-control/index.md) 
> [基于角色的授权](../management/access-control/role-based-authorization.md) 

本文介绍了用于管理安全角色的控制命令。
安全角色定义哪些安全主体（用户和应用程序）有权对受保护的资源（例如数据库或表）进行操作，以及允许进行哪些操作。 例如，对于特定的数据库，具有 `database viewer` 安全角色的主体可以查询和查看该数据库的所有实体（受限制的表除外）。

安全角色可以与安全主体或安全组（可以有其他安全主体或其他安全组）相关联。 当安全主体尝试对受保护的资源进行操作时，系统将检查该主体是否与至少一个授权对资源执行此操作的安全角色相关联。 这称为 **授权检查** 。 授权检查失败会中止操作。

**语法**

安全角色管理命令的语法：

*Verb* *SecurableObjectType* *SecurableObjectName* *Role* [`(` *ListOfPrincipals* `)` [ *Description* ]]

* *Verb* 指示要执行的操作类型：`.show`、`.add`、`.drop` 和 `.set`。

    |*谓词* |说明                                  |
    |-------|---------------------------------------------|
    |`.show`|返回当前的一个或多个值。         |
    |`.add` |将一个或多个主体添加到角色。     |
    |`.drop`|从角色中删除一个或多个主体。|
    |`.set` |将角色设置为特定主体列表，并删除所有以前的主体（如果有）。|

* SecurableObjectType 是指定了其角色的对象的类型。

    |SecurableObjectType|说明|
    |---------------------|-----------|
    |`database`|指定的数据库|
    |`table`|指定的表|
    |`materialized-view`| 指定的[具体化视图](materialized-views/materialized-view-overview.md)| 

* SecurableObjectName 是对象的名称。

* Role 是相关角色的名称。

    |*角色*      |说明|
    |------------|-----------|
    |`principals`|只能作为 `.show` 谓词的一部分出现；返回可能影响安全对象的主体的列表。|
    |`admins`    |对安全对象具有控制权，包括查看、修改和删除对象及所有子对象的功能。|
    |`users`     |可以查看安全对象，并在其下创建新对象。|
    |`viewers`   |可以查看安全对象。|
    |`unrestrictedviewers`|仅在数据库级别允许查看受限制的表（这些表未向“普通”`viewers` 和 `users` 公开）。|
    |`ingestors` |仅在数据库级别允许将数据引入到所有表中。|
    |`monitors`  ||

* ListOfPrincipals 是可选的、以逗号分隔的安全主体标识符列表（`string` 类型的值）。

* Description 是与关联一起存储的 `string` 类型的可选值，用于将来的审核。

## <a name="example"></a>示例

以下控制命令列出对数据库中的 `StormEvents` 表具有某些访问权限的所有安全主体：

```kusto
.show table StormEvents principals
```

下面是此命令的可能结果：

|角色 |PrincipalType |PrincipalDisplayName |PrincipalObjectId |PrincipalFQN 
|---|---|---|---|---
|Apsty 数据库管理员 |Azure AD 用户 |Mark Smith |cd709aed-a26c-e3953dec735e |aaduser=msmith@fabrikam.com|

## <a name="managing-database-security-roles"></a>管理数据库安全角色

`.set` `database` *DatabaseName* *Role* `none` [`skip-results`]

`.set` `database` *DatabaseName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

`.add` `database` *DatabaseName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

`.drop` `database` *DatabaseName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

第一个命令从角色中删除所有主体。 第二个命令从角色中删除所有主体，并设置一组新的主体。 第三个命令向角色中添加新的主体，不删除现有主体。 最后一个命令从角色中删除指定的主体，并保留其他主体。

其中：

* DatabaseName 是要修改其安全角色的数据库的名称。

* Role 是 `admins`、`ingestors`、`monitors`、`unrestrictedviewers`、`users` 或 `viewers`。

* Principal 是一个或多个主体。 请参阅[主体和标识提供者](./access-control/principals-and-identity-providers.md)，了解如何指定这些主体。

* `skip-results`（如果已提供）会要求命令不返回更新的数据库主体列表。

* Description（如果已提供）是将与更改关联的文本，可通过相应的 `.show` 命令进行检索。

<!-- TODO: Need more examples for the public syntax. Until then we're keeping this internal -->


## <a name="managing-table-security-roles"></a>管理表安全角色

`.set` `table` *TableName* *Role* `none` [`skip-results`]

`.set` `table` *TableName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

`.add` `table` *TableName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

`.drop` `table` *TableName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

第一个命令从角色中删除所有主体。 第二个命令从角色中删除所有主体，并设置一组新的主体。 第三个命令向角色中添加新的主体，不删除现有主体。 最后一个命令从角色中删除指定的主体，并保留其他主体。

其中：

* TableName 是要修改其安全角色的表的名称。

* Role 是 `admins` 或 `ingestors`。

* Principal 是一个或多个主体。 请参阅[主体和标识提供者](./access-control/principals-and-identity-providers.md)，了解如何指定这些主体。

* `skip-results`（如果已提供）会要求命令不返回更新的表主体列表。

* Description（如果已提供）是将与更改关联的文本，可通过相应的 `.show` 命令进行检索。

**示例**

```kusto
.add table Test admins ('aaduser=imike@fabrikam.com ')
```

## <a name="managing-materialized-view-security-roles"></a>管理具体化视图安全角色

`.show` `materialized-view` *MaterializedViewName* `principals`

`.set` `materialized-view` *MaterializedViewName* `admins` `(` *Principal* `,[` *Principal...* `])`

`.add` `materialized-view` *MaterializedViewName* `admins` `(` *Principal* `,[` *Principal...* `])`

`.drop` `materialized-view` *MaterializedViewName* `admins` `(` *Principal* `,[` *Principal...* `])`

其中：

* MaterializedViewName 是要修改其安全角色的具体化视图的名称
* Principal 是一个或多个主体。 请参阅[主体和标识提供者](./access-control/principals-and-identity-providers.md)

## <a name="managing-function-security-roles"></a>管理函数安全角色

`.set` `function` *FunctionName* *Role* `none` [`skip-results`]

`.set` `function` *FunctionName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

`.add` `function` *FunctionName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

`.drop` `function` *FunctionName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [ *Description* ]

第一个命令从角色中删除所有主体。 第二个命令从角色中删除所有主体，并设置一组新的主体。 第三个命令向角色中添加新的主体，不删除现有主体。 最后一个命令从角色中删除指定的主体，并保留其他主体。

其中：

* FunctionName 是要修改其安全角色的函数的名称。

* Role 始终为 `admin`。

* Principal 是一个或多个主体。 请参阅[主体和标识提供者](./access-control/principals-and-identity-providers.md)，了解如何指定这些主体。

* `skip-results`（如果已提供）会要求命令不返回更新的函数主体列表。

* Description（如果已提供）是将与更改关联的文本，可通过相应的 `.show` 命令进行检索。

**示例**

```kusto
.add function MyFunction admins ('aaduser=imike@fabrikam.com') 'This user should have access'
```

