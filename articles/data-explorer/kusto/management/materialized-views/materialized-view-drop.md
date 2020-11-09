---
title: 删除具体化视图 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 drop materialized view 命令。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: 26ae92f16a6d869599d170c691d31228b689ddfb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106574"
---
# <a name="drop-materialized-view"></a>.drop materialized-view 

删除具体化视图。

需要[数据库管理员](../access-control/role-based-authorization.md)或具体化视图管理员权限。

## <a name="syntax"></a>语法

`.drop` `materialized-view` *MaterializedViewName*

## <a name="properties"></a>属性

| properties | 类型| 说明 |
|----------------|-------|-----|
| MaterializedViewName| 字符串| 具体化视图的名称。|

## <a name="returns"></a>返回

此命令返回数据库中的其余具体化视图，这是 [show materialized view](materialized-view-show-commands.md#show-materialized-view) 命令的输出。

## <a name="example"></a>示例

```kusto
.drop materialized-view ViewName
```

## <a name="output"></a>输出

|输出参数 |类型 |说明
|---|---|---|
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
|EffectiveDateTime|datetime|视图的生效日期时间，在创建期间确定（请参阅 [.create materialized-view](materialized-view-create.md#create-materialized-view)）
