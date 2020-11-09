---
title: 表管理 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的表管理。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/27/2020
ms.date: 10/29/2020
ms.openlocfilehash: e5387e28c4b8c4aa18609686122fa9056cd7064d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104276"
---
# <a name="tables-management"></a>表管理

本主题讨论表格的生命周期和可帮助浏览、创建和更改表格的相关控制命令。

请选择下表中的链接，以获取相关详细信息。

| 命令                                                                                                                 | 操作                       |
|--------------------------------------------------------------------------------------------------------------------------|---------------------------------|
| [`.alter table docstring`](alter-table-docstring-command.md), [`.alter table folder`](alter-table-folder-command.md)                                                                                                                                                                                                   | 管理表显示属性 |
| [`.create ingestion mapping`](create-ingestion-mapping-command.md), [`.show ingestion mappings`](show-ingestion-mapping-command.md), [`.alter ingestion mapping`](alter-ingestion-mapping-command.md), [`.drop ingestion mapping`](drop-ingestion-mapping-command.md)                                                                    | 管理引入映射        |
| [`.create tables`](create-tables-command.md), [`.create table`](create-table-command.md), [`.alter table`](alter-table-command.md), [`.alter-merge table`](alter-table-command.md), [`.drop tables`](drop-table-command.md), [`.drop table`](drop-table-command.md), [`.undo drop table`](undo-drop-table-command.md), [`.rename table`](rename-table-command.md) | 创建/修改/删除表       |
| [`.show tables`](show-tables-command.md) [`.show table details`](show-table-details-command.md)[`.show table schema`](show-table-schema-command.md)                                                                                      | 枚举数据库中的表  |
| `.ingest`、`.set`、`.append`、`.set-or-append`（有关详细信息，请参阅[数据引入](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands)）。                                                                                                                                                                                      | 将数据引入到表中     |
| [`.clear table data`](clear-table-data-command.md)                            | 清除表中的所有数据  |

## <a name="crud-naming-conventions-for-tables"></a>表的 CRUD 命名约定 
（有关完整详细信息，请参阅上表中的链接部分。）
 
| 命令语法                             | 语义                                                                                                             |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| `.create entityType entityName ...`        | 如果该类型和名称的实体存在，则返回该实体。 否则，创建该实体。                          |
| `.create-merge entityType entityName...`   | 如果该类型和名称的实体存在，则将现有实体与指定的实体合并。 否则，创建该实体。 |
| `.alter entityType entityName ...`         | 如果该类型和名称的实体不存在，则会出错。 否则，请将其替换为指定的实体。            |
| `.alter-merge entityType entityName ...`   | 如果该类型和名称的实体不存在，则会出错。 否则，请将其与指定的实体合并。              |
| `.drop entityType entityName ...`          | 如果该类型和名称的实体不存在，则会出错。 否则，请将其删除。                                         |
| `.drop entityType entityName ifexists ...` | 如果该类型和名称的实体不存在，则返回。 否则，请将其删除。                                        |
 
> [!NOTE]
> “合并”是两个实体的逻辑合并：
>
> * 如果一个属性是为一个实体定义的，而不是为另一个实体定义的，则该属性将与其原始值一起出现在合并的实体中。
> * 如果一个属性是为两个实体定义的，并且在这两个实体中具有相同的值，则该属性与该值一起在合并的实体中出现一次。
> * 如果一个属性是为两个实体定义的，但具有不同的值，则会引发错误。