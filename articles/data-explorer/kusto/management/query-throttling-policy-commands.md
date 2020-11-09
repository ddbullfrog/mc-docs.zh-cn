---
title: 查询限制策略命令 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的查询限制策略命令
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: miwalia
ms.service: data-explorer
ms.topic: reference
origin.date: 10/05/2020
ms.date: 10/30/2020
ms.openlocfilehash: fc387b61b224b8c428d7a266b04ff83de0b04fa9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106783"
---
# <a name="query-throttling-policy-commands"></a>查询限制策略命令

[查询限制策略](query-throttling-policy.md)是群集级别的策略，用于限制群集中的查询并发性。 这些查询限制策略命令需要 [AllDatabasesAdmin](../management/access-control/role-based-authorization.md) 权限。

## <a name="query-throttling-policy-object"></a>查询限制策略对象

群集可能已定义零个或一个查询限制策略。

当群集未定义查询限制策略时，将应用默认策略。 有关默认策略的详细信息，请参阅[查询限制](../concepts/querylimits.md)。

|properties  |类型    |说明                                                       |
|----------|--------|------------------------------------------------------------------|
|IsEnabled |`bool`  |说明查询限制策略是已启用 (true) 还是已禁用 (false)。     |
|MaxQuantity|`int`|群集可以运行的并发查询数。 数字必须具有正值。 |

## `.show cluster policy querythrottling`

返回群集的[查询限制策略](query-throttling-policy.md)。

### <a name="syntax"></a>语法

`.show` `cluster` `policy` `querythrottling`

### <a name="returns"></a>返回

返回包含以下列的表：

|列    |类型    |说明
|---|---|---
|PolicyName| string |策略名称 - QueryThrottlingPolicy
|EntityName| string |空
|策略    | string |一个用于定义查询限制策略的 JSON 对象，格式设置为[查询限制策略对象](#query-throttling-policy-object)

### <a name="example"></a>示例

<!-- csl -->
```
.show cluster policy querythrottling 
```

返回：

|PolicyName|EntityName|策略|ChildEntities|EntityType|
|---|---|---|---|---|
|QueryThrottlingPolicy||{"IsEnabled": true,"MaxQuantity":25}

## `.alter cluster policy querythrottling`

设置指定表的[查询限制策略](query-throttling-policy.md)。 

### <a name="syntax"></a>语法

`.alter` `cluster` `policy` `querythrottling` *QueryThrottlingPolicyObject*

### <a name="arguments"></a>参数

QueryThrottlingPolicyObject 是一个已定义查询限制策略对象的 JSON 对象。

### <a name="returns"></a>返回

设置群集查询限制策略对象，替代任何已定义的当前策略，然后返回相应的 [.show cluster policy `querythrottling`](#show-cluster-policy-querythrottling) 命令的输出。

### <a name="example"></a>示例

<!-- csl -->
```
.alter cluster policy querythrottling '{"IsEnabled": true, "MaxQuantity": 25}'
```

|PolicyName|EntityName|策略|ChildEntities|EntityType|
|---|---|---|---|---|
|QueryThrottlingPolicy||{"IsEnabled": true,"MaxQuantity":25}

## `.delete cluster policy querythrottling`

删除群集[查询限制策略](query-throttling-policy.md)对象。

### <a name="syntax"></a>语法

`.delete` `cluster` `policy` `querythrottling`
