---
title: Kusto 更新策略管理 - Azure 数据资源管理器
description: 了解 Azure 数据资源管理器中的更新策略命令。 了解如何显示、设置、更改和删除表更新策略。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 08/04/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1752762fa9e19e351c9cdb41fe0d447011f40cd4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105694"
---
# <a name="update-policy-commands"></a>更新策略命令

[更新策略](updatepolicy.md)是一个表级别策略对象，它自动运行查询，然后在将数据引入到另一个表时引入结果。

## <a name="show-update-policy"></a>显示更新策略

如果将 `*` 用作表名称，此命令将返回指定表或默认数据库中所有表的更新策略。

### <a name="syntax"></a>语法

* `.show` `table` TableName `policy` `update`
* `.show` `table` DatabaseName`.`TableName `policy` `update` 
* `.show` `table` `*` `policy` `update`

### <a name="returns"></a>返回

此命令返回每个表都有一条记录的表。

|列    |类型    |说明                                                                                                                                                           |
|----------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|EntityName|`string`|对其定义更新策略的实体的名称                                                                                                                |
|策略  |`string`|一个 JSON 数组，指示为该实体定义的所有更新策略，格式设置为[更新策略对象](updatepolicy.md#the-update-policy-object)|

### <a name="example"></a>示例

```kusto
.show table DerivedTableX policy update 
```

|EntityName        |策略                                                                                                                                    |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|[TestDB].[DerivedTableX]|[{"IsEnabled": true, "Source":"MyTableX","Query":"MyUpdateFunction()","IsTransactional": false,"PropagateIngestionProperties": false}]|

## <a name="alter-update-policy"></a>更改更新策略

此命令设置指定表的更新策略。

### <a name="syntax"></a>语法

* `.alter` `table` TableName `policy` `update` ArrayOfUpdatePolicyObjects 
* `.alter` `table` DatabaseName`.`TableName `policy` `update` ArrayOfUpdatePolicyObjects  

ArrayOfUpdatePolicyObjects 是一个定义了零个或多个更新策略对象的 JSON 数组。

> [!NOTE]
> * 对更新策略对象的 `Query` 属性使用存储的函数。
   只需修改函数定义，而不是整个策略对象。
> * 如果 `IsEnabled` 设置为 `true`，则在设置更新策略时对其执行以下验证：
>    * `Source` - 检查目标数据库中是否存在该表。
>    * `Query` 
>        * 检查架构定义的架构是否与目标表中的架构匹配
>        * 检查查询是否引用更新策略的 `source` 表。 
        可以通过在 with 属性中设置 `AllowUnreferencedSourceTable=true` 来定义不引用源的更新策略查询（请参见下面的示例），但是由于性能问题，不建议使用此策略。 每次将数据引入到源表时，更新策略执行起来会考虑不同表中的所有记录。
 >       * 检查策略是否不会导致在目标数据库中的更新策略链中创建循环。
 > * 如果 `IsTransactional` 设置为 `true`，则检查 `TableAdmin` 权限是否也针对 `Source`（源表）进行了验证。
 > * 测试更新策略或函数的性能，然后应用它以在源表每次引入时运行。 有关详细信息，请参阅[测试更新策略的性能影响](updatepolicy.md#performance-impact)。

### <a name="returns"></a>返回

此命令设置表的更新策略对象，替代所有当前策略，然后返回相应的 [.show table update policy](#show-update-policy) 命令的输出。

### <a name="example"></a>示例

```kusto
// Create a function that will be used for update
.create function 
MyUpdateFunction()
{
    MyTableX
    | where ColumnA == 'some-string'
    | summarize MyCount=count() by ColumnB, Key=ColumnC
    | join (OtherTable | project OtherColumnZ, Key=OtherColumnC) on Key
    | project ColumnB, ColumnZ=OtherColumnZ, Key, MyCount
}

// Create the target table (if it doesn't already exist)
.set-or-append DerivedTableX <| MyUpdateFunction() | limit 0

// Use update policy on table DerivedTableX
.alter table DerivedTableX policy update
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyUpdateFunction()", "IsTransactional": false, "PropagateIngestionProperties": false}]'
```

* 当发生对源表（在本例中为 `MyTableX`）的引入时，在该表中创建一个或多个盘区（数据分片）
* 在更新策略对象（在本例中为 `MyUpdateFunction()`）中定义的 `Query` 将仅在这些区上运行，而不会在整个表上运行。
  * “范围界定”是在内部自动完成的，在定义 `Query` 时不应进行处理。
  * 当引入到 `DerivedTableX` 派生表时，仅考虑每个引入操作中不同的新引入记录。

```kusto
// The following example will throw an error for not referencing the source table in the update policy query
// The policy's source table is MyTableX, whereas the query only references MyOtherTable. 
.alter table DerivedTableX policy update
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyOtherTable", "IsTransactional": false, "PropagateIngestionProperties": false}]'

// Adding the following properties will suppress the error (but is not recommended)
.alter table DerivedTableX policy update with (AllowUnreferencedSourceTable=true)
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyOtherTable", "IsTransactional": false, "PropagateIngestionProperties": false}]'

```

## <a name="alter-merge-table-tablename-policy-update"></a>.alter-merge table TableName policy update

此命令修改指定表的更新策略。

**语法**

* `.alter-merge` `table` TableName `policy` `update` ArrayOfUpdatePolicyObjects 
* `.alter-merge` `table` DatabaseName`.`TableName `policy` `update` ArrayOfUpdatePolicyObjects  

ArrayOfUpdatePolicyObjects 是一个定义了零个或多个更新策略对象的 JSON 数组。

> [!NOTE]
> * 对批量实现更新策略对象的查询属性使用存储的函数。 
     只需修改函数定义，不需修改整个策略对象。
> * 该验证与对 `alter` 命令执行的验证相同。

**返回**

此命令追加到表的更新策略对象，替代所有当前策略，然后返回相应的 [.show table TableName update policy](#show-update-policy) 命令的输出。

**示例**

```kusto
.alter-merge table DerivedTableX policy update 
@'[{"IsEnabled": true, "Source": "MyTableY", "Query": "MyUpdateFunction()", "IsTransactional": false}]'  
``` 

## <a name="delete-table-tablename-policy-update"></a>.delete table TableName policy update

删除指定表的更新策略。

**语法**

* `.delete` `table` TableName `policy` `update`
* `.delete` `table` DatabaseName`.`TableName `policy` `update` 

**返回**

此命令删除表的更新策略对象，然后返回相应的 [.show table TableName update policy](#show-update-policy) 命令的输出。

**示例**

```kusto
.delete table DerivedTableX policy update 
```
