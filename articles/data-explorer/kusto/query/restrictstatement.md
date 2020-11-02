---
title: Restrict 语句 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 Restrict 语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 7b072a058d9a9a896d3fb2e166d67c5068869514
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105018"
---
# <a name="restrict-statement"></a>Restrict 语句

::: zone pivot="azuredataexplorer"

Restrict 语句限制一组表/视图实体，这些实体对其后的查询语句可见。 例如，在包含两个表（`A`、`B`）的数据库中，应用程序可以防止查询的其余部分访问 `B`，并且只能通过使用视图“查看”表 `A` 的有限形式。

Restrict 语句主要适用于中间层应用程序，这些应用程序接受来自用户的查询并希望对这些查询应用行级安全机制。 中间层应用程序可以在用户的查询前加上逻辑模型，这是一组 let 语句，用于定义视图以限制用户对数据的访问权限（例如 `T | where UserId == "..."`）。 作为要添加的最后一个语句，它会将用户限制为只能访问该逻辑模型。

> [!NOTE]
> Restrict 语句可用于限制对另一个数据库或群集中的实体的访问（群集名称不支持通配符）。

## <a name="syntax"></a>语法

`restrict` `access` `to` `(` [ *EntitySpecifier* [`,` ...]] `)`

其中 EntitySpecifier 是以下值之一：
* 由 let 语句定义为表格视图的标识符。
* 表引用（类似于 union 语句使用的引用）。
* 模式声明定义的模式。

所有并非由 restrict 语句指定的表、表格视图或模式将对查询的其余部分变为“不可见”。 

## <a name="arguments"></a>参数

Restrict 语句可以获取一个或多个参数，这些参数定义实体名称解析期间的许可限制。 实体可以是：
* 出现在 `restrict` 语句前面的 [let 语句](./letstatement.md)。 

  ```kusto
  // Limit access to 'Test' let statement only
  let Test = () { print x=1 };
  restrict access to (Test);
  ```

* 在数据库元数据中定义的[表](../management/tables.md)或[函数](../management/functions.md)。

    ```kusto
    // Assuming the database that the query uses has table Table1 and Func1 defined in the metadata, 
    // and other database 'DB2' has Table2 defined in the metadata
    
    restrict access to (database().Table1, database().Func1, database('DB2').Table2);
    ```

* 可以匹配多个 [let 语句](./letstatement.md)或表/函数的通配符模式  

    ```kusto
    let Test1 = () { print x=1 };
    let Test2 = () { print y=1 };
    restrict access to (*);
    // Now access is restricted to Test1, Test2 and no tables/functions are accessible.

    // Assuming the database that the query uses has table Table1 and Func1 defined in the metadata.
    // Assuming that database 'DB2' has table Table2 and Func2 defined in the metadata
    restricts access to (database().*);
    // Now access is restricted to all tables/functions of the current database ('DB2' is not accessible).

    // Assuming the database that the query uses has table Table1 and Func1 defined in the metadata.
    // Assuming that database 'DB2' has table Table2 and Func2 defined in the metadata
    restricts access to (database('DB2').*);
    // Now access is restricted to all tables/functions of the database 'DB2'
    ```

## <a name="examples"></a>示例

下面的示例演示了中间层应用程序如何在用户的查询前面加上一个逻辑模型，以防止用户查询任何其他用户的数据。

```kusto
// Assume the database has a single table, UserData,
// with a column called UserID and other columns that hold
// per-user private information.
//
// The middle-tier application generates the following statements.
// Note that "username@domain.com" is something the middle-tier application
// derives per-user as it authenticates the user.
let RestrictedData = view () { Data | where UserID == "username@domain.com" };
restrict access to (RestrictedData);
// The rest of the query is something that the user types.
// This part can only reference RestrictedData; attempting to reference Data
// will fail.
RestrictedData | summarize IrsLovesMe=sum(Salary) by Year, Month
```

```kusto
// Restricting access to Table1 in the current database (database() called without parameters)
restrict access to (database().Table1);
Table1 | count

// Restricting access to Table1 in the current database and Table2 in database 'DB2'
restrict access to (database().Table1, database('DB2').Table2);
union 
    (Table1),
    (database('DB2').Table2))
| count

// Restricting access to Test statement only
let Test = () { range x from 1 to 10 step 1 };
restrict access to (Test);
Test
 
// Assume that there is a table called Table1, Table2 in the database
let View1 = view () { Table1 | project Column1 };
let View2 = view () { Table2 | project Column1, Column2 };
restrict access to (View1, View2);
 
// When those statements appear before the command - the next works
let View1 = view () { Table1 | project Column1 };
let View2 = view () { Table2 | project Column1, Column2 };
restrict access to (View1, View2);
View1 |  count
 
// When those statements appear before the command - the next access is not allowed
let View1 = view () { Table1 | project Column1 };
let View2 = view () { Table2 | project Column1, Column2 };
restrict access to (View1, View2);
Table1 |  count
```

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
