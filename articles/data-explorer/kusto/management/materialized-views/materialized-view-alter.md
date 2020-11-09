---
title: 更改具体化视图 - Azure 数据资源管理器
description: 本文介绍如何在 Azure 数据资源管理器中更改具体化视图。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: 47c5a5de623a19696ca9e5a482967508bb4fb7fd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106654"
---
# <a name="alter-materialized-view"></a>.alter materialized-view

更改[具体化视图](materialized-view-overview.md)可用于更改具体化视图的查询，同时保留视图中的现有数据。

需要[数据库管理员](../access-control/role-based-authorization.md)权限或具体化视图的管理员角色。

> [!WARNING]
> 更改具体化视图时要格外小心。 使用不当可能会导致数据丢失。

## <a name="syntax"></a>语法

`.alter` `materialized-view`  
[ `with` `(`*PropertyName* `=` *PropertyValue*`,`...`)`]  
*ViewName* `on table` *SourceTableName*  
`{`  
    &nbsp;&nbsp;&nbsp;&nbsp;*Query*  
`}`

## <a name="arguments"></a>参数

|参数|类型|说明
|----------------|-------|---|
|视图名|字符串|具体化视图名称。|
|SourceTableName|字符串|定义视图的源表的名称。|
|查询|String|具体化视图查询。|

## <a name="properties"></a>属性

`dimensionTables` 是 materialized-view alter 命令唯一支持的属性。 此属性应在查询引用维度表时使用。 有关详细信息，请参阅 [.create materialized-view](materialized-view-create.md) 命令。

## <a name="use-cases"></a>用例

* 向视图添加聚合 - 例如，通过将视图查询更改为 `T | summarize count(), min(Value), avg(Value) by Id`，将 `avg` 聚合添加到 `T | summarize count(), min(Value) by Id` 中。
* 更改除 summarize 运算符之外的运算符。 例如，通过将 `T | summarize arg_max(Timestamp, *) by User` 更改为 `T | where User != 'someone' | summarize arg_max(Timestamp, *) by User` 来筛选掉某些记录。
* 由于源表发生了更改，在不更改查询的情况下更改。 例如，假定 `T | summarize arg_max(Timestamp, *) by Id` 视图，它未设置为 `autoUpdateSchema`（请参阅 [.create materialized-view](materialized-view-create.md) 命令）。 如果在视图的源表中添加或删除了某列，则该视图将自动禁用。 使用完全相同的查询执行 alter 命令，以更改具体化视图的架构，使之与新表架构保持一致。 在更改后，仍必须使用 [enable materialized view](materialized-view-enable-disable.md) 命令显式启用该视图。

## <a name="alter-materialized-view-limitations"></a>更改具体化视图的限制

* **不支持的更改：**
    * 不支持更改列类型。
    * 不支持重命名列。 例如，将 `T | summarize count() by Id` 视图更改为 `T | summarize Count=count() by Id` 将删除 `count_` 列并创建新列 `Count`（该列最初将仅包含 null）。
    * 不支持通过表达式对具体化视图组进行更改。

* **对现有数据的影响：**
    * 更改具体化视图不会影响现有数据。
    * 新列将接收所有现有记录的空值，直到引入的记录发布 alter 命令来修改 null 值。
        * 例如：如果 `T | summarize count() by bin(Timestamp, 1d)` 视图更改为 `T | summarize count(), sum(Value) by bin(Timestamp, 1d)`，则对于那些已在更改视图之前处理的记录的特定 `Timestamp=T`，`sum` 列将包含部分数据。 此视图仅包含在 alter 执行后处理的记录。
    * 向查询添加筛选器不会更改已具体化的记录。 筛选器将仅适用于新引入的记录。
