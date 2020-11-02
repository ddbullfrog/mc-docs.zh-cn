---
title: 查询结果缓存 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的查询结果缓存功能。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: amitof
ms.service: data-explorer
ms.topic: reference
origin.date: 06/16/2020
ms.date: 10/29/2020
ms.openlocfilehash: 2d02717f63eff91a214d1f3019242b9fbff90152
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104022"
---
# <a name="query-results-cache"></a>查询结果缓存

Kusto 提供了查询结果缓存。 发出查询时，你可以选择获取缓存的结果。 如果查询结果可以由缓存返回，你可以体验到更好的查询性能和更低的资源消耗。 然而，实现此性能的代价是结果有一些“过时”。

## <a name="use-the-cache"></a>使用缓存

将 `query_results_cache_max_age` 选项设置为查询的一部分可使用查询结果缓存。 你可以在查询文本中设置此选项，也可以将其作为客户端请求属性进行设置。 例如：

```kusto
set query_results_cache_max_age = time(5m);
GithubEvent
| where CreatedAt > ago(180d)
| summarize arg_max(CreatedAt, Type) by Id
```

选项值为一个 `timespan`，它指示结果缓存的最长时间（从查询开始时间算起）。 超过所设置的时间跨度之后，缓存条目会过时，不会再次使用。 将值设置为 0 等效于不设置此选项。

## <a name="compatibility-between-queries"></a>查询之间的兼容性

### <a name="identical-queries"></a>相同的查询

查询结果缓存仅为被认为与前一个缓存查询“相同”的查询返回结果。 如果满足以下所有条件，则将两个查询视为相同的查询：

* 两个查询具有相同的表示形式（表示为 UTF-8 字符串）。
* 两个查询是针对同一数据库发出的。
* 两个查询具有相同的[客户端请求属性](../api/netfx/request-properties.md)。 对于缓存用途，将忽略下列属性：
   * [ClientRequestId](../api/netfx/request-properties.md#the-clientrequestid-x-ms-client-request-id-named-property)
   * [应用程序](../api/netfx/request-properties.md#the-application-x-ms-app-named-property)
   * [用户](../api/netfx/request-properties.md#the-user-x-ms-user-named-property)

### <a name="incompatible-queries"></a>不兼容的查询

如果满足以下任一条件，则不会缓存查询结果：
 
* 查询引用启用了 [RestrictedViewAccess](../management/restrictedviewaccesspolicy.md) 策略的表。
* 查询引用启用了 [RowLevelSecurity](../management/rowlevelsecuritypolicy.md) 策略的表。
* 查询使用以下任一函数：
    * [current_principal](current-principalfunction.md)
    * [current_principal_details](current-principal-detailsfunction.md)
    * [current_principal_is_member_of](current-principal-ismemberoffunction.md)
* 查询访问[外部表](schema-entities/externaltables.md)或[外部数据](externaldata-operator.md)。
* 查询使用 [evaluate 插件](evaluateoperator.md)运算符。

## <a name="no-valid-cache-entry"></a>不存在有效的缓存条目

如果找不到满足时间约束的缓存结果，或者缓存中不存在来自“完全相同”查询的缓存结果，则会执行该查询，并会在满足以下条件时缓存其结果： 

* 查询执行成功完成，并且
* 查询结果大小未超过 16 MB。

## <a name="results-from-the-cache"></a>缓存中的结果

服务如何指示查询结果是从缓存中提供的？
当响应查询时，Kusto 会发送一个额外的 [ExtendedProperties](../api/rest/response.md) 响应表，其中包括 `Key` 列和 `Value` 列。
缓存的查询结果会为该表追加一个额外的行：
* 该行的 `Key` 列将包含字符串 `ServerCache`
* 该行的 `Value` 列将包含具有两个字段的属性包：
   * `OriginalClientRequestId` - 指定原始请求的 [ClientRequestId](../api/netfx/request-properties.md#the-clientrequestid-x-ms-client-request-id-named-property)。
   * `OriginalStartedOn` - 指定原始请求的执行开始时间。

## <a name="distribution"></a>分布

群集节点不共享缓存。 每个节点的专用存储中都有一个专用缓存。 如果在不同节点上有两个完全相同的查询，则在这两个节点上都会执行并缓存查询。 如果使用[弱一致性](../concepts/queryconsistency.md)，则可能会发生此过程。

## <a name="management"></a>管理

支持以下管理和可观测性命令：

* [显示缓存](../management/show-query-results-cache-command.md)。
* [清除缓存](../management/clear-query-results-cache-command.md)。

## <a name="capacity"></a>容量

缓存容量目前固定为每个群集节点 1 GB。
逐出策略是 LRU。
