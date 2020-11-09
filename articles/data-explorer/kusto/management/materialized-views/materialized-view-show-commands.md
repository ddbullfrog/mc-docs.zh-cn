---
title: show materialized view 命令 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 show materialized views 命令。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: 72dc2613194333a58e9ddcab485855b64dafe1d3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106758"
---
# <a name="show-materialized-views-commands"></a>.show materialized-views 命令

以下 show 命令显示有关[具体化视图](materialized-view-overview.md)的信息。

## <a name="show-materialized-view"></a>.show materialized-view

显示有关具体化视图的定义及其当前状态的信息。

### <a name="syntax"></a>语法

`.show` `materialized-view` *MaterializedViewName*

`.show` `materialized-views`

### <a name="properties"></a>属性

|properties|类型|说明
|----------------|-------|---|
|MaterializedViewName|字符串|具体化视图的名称。|

### <a name="example"></a>示例

```kusto
.show materialized-view ViewName
```

### <a name="output"></a>输出

|输出参数 |类型 |说明
|---|---|---
|名称  |String |具体化视图的名称。
|SourceTable|字符串|具体化视图的源表。
|查询|String|具体化视图查询。
|MaterializedTo|datetime|源表中的最大具体化 ingestion_time() 时间戳。 有关详细信息，请参阅[具体化视图的工作原理](materialized-view-overview.md#how-materialized-views-work)。
|LastRun|datetime |上次运行具体化的时间。
|LastRunResult|String|上次运行的结果。 如果运行成功，则返回 `Completed`，否则返回 `Failed`。
|IsHealthy|bool|当视图被认为正常时为 `True`，否则为 `False`。 如果视图在最后一小时之前被成功地具体化（`MaterializedTo` 大于 `ago(1h)`），则认为它是正常的。
|IsEnabled|bool|如果视图已启用，则为 `True`（请参阅[禁用或启用具体化视图](materialized-view-enable-disable.md)）。
|文件夹|string|具体化视图文件夹。
|DocString|string|具体化视图的文档字符串。
|AutoUpdateSchema|bool|视图是否已启用自动更新。
|EffectiveDateTime|datetime|视图的生效日期时间，在创建期间确定（请参阅 [.create materialized-view](materialized-view-create.md#create-materialized-view)）。

## <a name="show-materialized-view-schema"></a>.show materialized-view schema

以 CSL/JSON 格式返回具体化视图的架构。

### <a name="syntax"></a>语法

`.show` `materialized-view` *MaterializedViewName* `cslschema`

`.show` `materialized-view` *MaterializedViewName* `schema` `as` `json`

`.show` `materialized-view` *MaterializedViewName* `schema` `as` `csl`

### <a name="output-parameters"></a>输出参数

| 输出参数 | 类型   | 说明                                               |
|------------------|--------|-----------------------------------------------------------|
| TableName        | 字符串 | 具体化视图的名称。                        |
| 架构           | 字符串 | 具体化视图 csl 架构                          |
| DatabaseName     | String | 具体化视图所属的数据库       |
| 文件夹           | String | 具体化视图的文件夹                                |
| DocString        | String | 具体化视图的 docstring                             |

## <a name="show-materialized-view-extents"></a>.show materialized-view extents

返回具体化视图的“具体化”部分中的区。 有关“具体化”部分的定义，请参阅[具体化视图的工作原理](materialized-view-overview.md#how-materialized-views-work)。

此命令提供的详细信息与 [show table extents](../show-extents.md#table-level) 命令相同。

### <a name="syntax"></a>语法

`.show` `materialized-view` *MaterializedViewName* `extents` [`hot`]
 
## <a name="show-materialized-view-failures"></a>.show materialized-view failures

返回在具体化视图的具体化过程中发生的失败。

### <a name="syntax"></a>语法

`.show` `materialized-view` *MaterializedViewName* `failures`

### <a name="properties"></a>属性

|properties|类型|说明
|----------------|-------|---|
|MaterializedViewName|字符串|具体化视图的名称。|

### <a name="output"></a>输出

|输出参数 |类型 |说明
|---|---|---
|名称  |时间戳 |失败时间戳。
|OperationId  |String |失败的运行的操作 ID。
|名称|String|具体化视图名称。
|LastSuccessRun|datetime|成功完成的上次运行的时间戳。
|FailureKind|String|失败类型。
|详细信息|String|失败详细信息。

