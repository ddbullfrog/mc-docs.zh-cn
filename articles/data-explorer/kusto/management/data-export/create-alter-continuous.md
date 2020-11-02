---
title: 创建或更改连续数据导出 - Azure 数据资源管理器
description: 本文介绍了如何创建或更改 Azure 数据资源管理器中的连续数据导出。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/03/2020
ms.date: 10/29/2020
ms.openlocfilehash: 8c34939c5237fad7ae05fc684dcafa8b543946c0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104457"
---
# <a name="create-or-alter-continuous-export"></a>创建或更改连续导出

创建或更改连续导出作业。

## <a name="syntax"></a>语法

`.create-or-alter` `continuous-export` *ContinuousExportName* <br>
[ `over` `(`*T1* , *T2* `)`] <br>
`to` `table` *ExternalTableName* <br> [ `with` `(`*PropertyName* `=` *PropertyValue*`,`...`)`]<br>
\<| *Query*

## <a name="properties"></a>属性

| properties             | 类型     | 说明   |
|----------------------|----------|---------------------------------------|
| ContinuousExportName | String   | 连续导出的名称。 该名称在数据库中必须独一无二，用于定期运行连续导出。      |
| ExternalTableName    | String   | 要导出到的[外部表](../external-table-commands.md)的名称。  |
| 查询                | String   | 要导出的查询。  |
| over (T1, T2)        | String   | 查询中可选的以逗号分隔的事实数据表列表。 如果不指定此项，将假定查询中引用的所有表都是事实数据表。 如果指定此项，则不在此列表中的表将被视为维度表，并且将没有作用域（所有记录都将参与所有导出）。 有关详细信息，请参阅[连续数据导出概述](continuous-data-export.md)。 |
| intervalBetweenRuns  | Timespan | 连续导出执行之间的时间跨度。 必须大于 1 分钟。   |
| forcedLatency        | Timespan | 一个可选的时间段，将查询范围限定为在此时间段之前（相对于当前时间）引入的记录。 例如，如果查询执行一些聚合/联接操作，而你想要确保在运行导出之前已引入所有相关记录，则此属性很有用。

除此之外，连续导出的创建命令支持[导出到外部表命令](export-data-to-an-external-table.md)支持的所有属性。 

## <a name="example"></a>示例

```kusto
.create-or-alter continuous-export MyExport
over (T)
to table ExternalBlob
with
(intervalBetweenRuns=1h, 
 forcedLatency=10m, 
 sizeLimit=104857600)
<| T
```

| 名称     | ExternalTableName | 查询 | ForcedLatency | IntervalBetweenRuns | CursorScopedTables         | ExportProperties                   |
|----------|-------------------|-------|---------------|---------------------|----------------------------|------------------------------------|
| MyExport | ExternalBlob      | S     | 00:10:00      | 01:00:00            | [<br>  "['DB'].['S']"<br>] | {<br>  "SizeLimit":104857600<br>} |
