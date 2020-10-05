---
title: .show table details - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 .show table details。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/04/2020
ms.date: 09/24/2020
ms.openlocfilehash: 0b5dec233ee381f93639236ef1092469365f02ea
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146694"
---
# <a name="show-table-details"></a>.show table details
返回一个集合，其中包含指定的表或数据库中的所有表，以及每个表的属性的详细摘要。

需要[数据库查看者权限](../management/access-control/role-based-authorization.md)。

```kusto
.show table T1 details
.show tables (T1, ..., Tn) details
.show tables details
```

**输出**

| 输出参数           | 类型     | 说明                                                                                     |
|----------------------------|----------|-------------------------------------------------------------------------------------------------|
| `TableName`                | 字符串   | 表的名称。                                                                          |
| `DatabaseName`             | String   | 表所属的数据库。                                                         |
| `Folder`                   | String   | 表的文件夹。                                                                             |
| `DocString`                | String   | 一个用来记录表的字符串。                                                                 |
| `TotalExtents`             | Int64    | 表中的总盘区数。                                                       |
| `TotalExtentSize`          | Double   | 表中的区的总大小（压缩大小 + 索引大小，以字节为单位）。               |
| `TotalOriginalSize`        | Double   | 表中数据的原始总大小（以字节为单位）。                                        |
| `TotalRowCount`            | Int64    | 表中的总行数。                                                          |
| `HotExtents`               | Int64    | 存储在热缓存中的表中的总盘区数。                              |
| `HotExtentSize`            | Double   | 存储在热缓存中的表中的区的总大小（压缩大小 + 索引大小，以字节为单位）。 |
| `HotOriginalSize`          | Double   | 存储在热缓存中的表中数据的原始总大小（以字节为单位）。               |
| `HotRowCount`              | Int64    | 存储在热缓存中的表中的总行数。                                 |
| `AuthorizedPrincipals`     | String   | 表的已授权主体（序列化为 JSON）。                                          |
| `RetentionPolicy`          | String   | 表的有效`*` 保留策略（序列化为 JSON）。                                  |
| `CachingPolicy`            | String   | 表的有效`*` 缓存策略（序列化为 JSON）。                                    |
| `ShardingPolicy`           | String   | 表的有效`*` 分片策略（序列化为 JSON.66666666666666）                     |
| `MergePolicy`              | String   | 表的有效`*` 合并策略（序列化为 JSON）。                                      |
| `StreamingIngestionPolicy` | String   | 表的有效`*` 流式引入策略（序列化为 JSON）。                        |
| `MinExtentsCreationTime`   | DateTime | 表中盘区的最小创建时间（如果没有盘区，则为 NULL）。         |
| `MaxExtentsCreationTime`   | DateTime | 表中盘区的最大创建时间（如果没有盘区，则为 NULL）。         |
| `RowOrderPolicy`           | String   | 表的有效行顺序策略（序列化为 JSON）。                                     |

`*`考虑父实体（例如数据库/群集）的策略。

**输出示例**

| TableName         | DatabaseName | 文件夹 | DocString | TotalExtents | TotalExtentSize | TotalOriginalSize | TotalRowCount | HotExtents | HotExtentSize | HotOriginalSize | HotRowCount | AuthorizedPrincipals                                                                                                                                                                               | RetentionPolicy                                                                                                                                       | CachingPolicy                                                                        | ShardingPolicy                                                                    | MergePolicy                                                                                                                                             | StreamingIngestionPolicy | MinExtentsCreationTime      | MaxExtentsCreationTime      |
|-------------------|--------------|--------|-----------|--------------|-----------------|-------------------|---------------|------------|---------------|-----------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-----------------------------|-----------------------------|
| 操作        | 操作   |        |           | 1164         | 37687203        | 53451358          | 223325        | 29         | 838752        | 1388213         | 5117        | [{"Type":"AAD User", "DisplayName":"My Name (upn: alias@fabrikam.com)", "ObjectId": "a7a77777-4c21-4649-95c5-350bf486087b", "FQN": "aaduser=a7a77777-4c21-4649-95c5-350bf486087b", "Notes": ""}] | {"SoftDeletePeriod":"365.00:00:00", "ContainerRecyclingPeriod":"1.00:00:00", "ExtentsDataSizeLimitInBytes":0, "OriginalDataSizeLimitInBytes":0 }  | { "DataHotSpan":"4.00:00:00", "IndexHotSpan":"4.00:00:00", "ColumnOverrides": [] } | { "MaxRowCount":750000, "MaxExtentSizeInMb":1024, "MaxOriginalSizeInMb":2048 } | { "RowCountUpperBoundForMerge":0, "MaxExtentsToMerge":100, "LoopPeriod":"01:00:00", "MaxRangeInHours":3, "AllowRebuild": true, "AllowMerge": true } | Null                     |
| ServiceOperations | 操作   |        |           | 1109         | 76588803        | 91553069          | 110125        | 27         | 2635742       | 2929926         | 3162        | [{"Type":"AAD User", "DisplayName":"My Name (upn: alias@fabrikam.com)", "ObjectId": "a7a77777-4c21-4649-95c5-350bf486087b", "FQN": "aaduser=a7a77777-4c21-4649-95c5-350bf486087b", "Notes": ""}] | { "SoftDeletePeriod":"365.00:00:00", "ContainerRecyclingPeriod":"1.00:00:00", "ExtentsDataSizeLimitInBytes":0, "OriginalDataSizeLimitInBytes":0 } | { "DataHotSpan":"4.00:00:00", "IndexHotSpan":"4.00:00:00", "ColumnOverrides": [] } | { "MaxRowCount":750000, "MaxExtentSizeInMb":1024, "MaxOriginalSizeInMb":2048 } | { "RowCountUpperBoundForMerge":0, "MaxExtentsToMerge":100, "LoopPeriod":"01:00:00", "MaxRangeInHours":3, "AllowRebuild": true, "AllowMerge": true } | Null                     | 2018-02-08 15:30:38.8489786 | 2018-02-14 07:47:28.7660267 |
