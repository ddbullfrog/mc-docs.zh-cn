---
title: 架构设计的最佳做法 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中架构设计的最佳做法。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/18/2020
ms.date: 08/18/2020
ms.openlocfilehash: 42466a74d71da6f2ca9502e054e10dff48bca5c6
ms.sourcegitcommit: 0c05bea089932ba8677b771cf4407a1fe3545fff
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061850"
---
# <a name="best-practices-for-schema-design"></a>针对架构设计的最佳做法

下面是要遵循的一些最佳做法。 它们可以提高管理命令的效率，对服务资源的影响较小。

|操作  |用途  |不要使用 | 注释 |
|---------|---------|---------|----
| **创建多个表**    |  使用单个 [`.create tables`](create-tables-command.md) 命令       | 不要发出多个 `.create table` 命令        | |
| **重命名多个表**    | 对 [`.rename tables`](rename-table-command.md) 进行一次调用        |  不要为每对表发出单独的调用   |    |
|**显示命令**   |   使用范围最小的 `.show` 命令 |   不要在竖线 (`|`) 之后应用筛选器   </ul></li>  | 尽量限制使用。 如果可能，请缓存这些命令返回的信息。 |
| 显示区  | 使用 `.show table T extents`   |不要使用 `.show cluster extents | where TableName == 'T'`  |
|  显示数据库架构。 |使用 `.show database DB schema`  |  不要使用 `.show schema | where DatabaseName == 'DB'` |
| **显示具有大型架构的群集上的架构** <br> |使用 [`.show databases schema`](../management/show-schema-database.md) |不要使用 `.show schema`| 例如，在数据库超过 100 个的群集上使用。
| **检查表是否存在或获取表的架构**|使用 `.show table T schema as json`|不要使用 `.show table T` |仅使用此命令来获取单个表的实际统计信息。|
| **定义将包含 `datetime` 值的表的架构**  |将相关列设置为 `datetime` 类型 | 请勿在查询时将 `string` 或数值列转换为 `datetime` 进行筛选（如果这可以在引入之前或引入过程中完成）|
| **向元数据添加区标记** |少量使用 |避免使用 `drop-by:` 标记，这些标记限制了系统在后台执行面向性能的整理过程的能力。|  <br> 请参阅[性能说明](../management/extents-overview.md#extent-tagging)。 |
