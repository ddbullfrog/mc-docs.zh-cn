---
title: cluster()（范围函数）- Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 cluster()（范围函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 17a597e529d7c83b02007cf1a935a6efe43843d2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104400"
---
# <a name="cluster-scope-function"></a>cluster()（范围函数）

::: zone pivot="azuredataexplorer"

将查询的引用更改为远程群集。 

```kusto
cluster('help').database('Sample').SomeTable
```

## <a name="syntax"></a>语法

`cluster(`*stringConstant*`)`

## <a name="arguments"></a>参数

* *stringConstant* ：被引用群集的名称。 群集名称可以是完全限定的 DNS 名称，也可以是以 `.kusto.chinacloudapi.cn` 为后缀的字符串。 在执行查询之前，参数必须是常量，即不能来自子查询求值。

**说明**

* 若要访问同一群集内的数据库，请使用 [database()](databasefunction.md) 函数。
* 有关跨群集查询和跨数据库查询的详细信息，请参阅[此文](cross-cluster-or-database-queries.md)  

## <a name="examples"></a>示例

### <a name="use-cluster-to-access-remote-cluster"></a>使用 cluster() 访问远程群集 

下一查询可在任何 Kusto 群集上运行。

```kusto
cluster('help').database('Samples').StormEvents | count

cluster('help.kusto.chinacloudapi.cn').database('Samples').StormEvents | count  
```

|计数|
|---|
|59066|

### <a name="use-cluster-inside-let-statements"></a>在 let 语句中使用 cluster() 

可以重写与上述查询相同的查询，以便使用可接收参数 `clusterName` 的内联函数（let 语句）- 该参数会传递到 cluster() 函数中。

```kusto
let foo = (clusterName:string)
{
    cluster(clusterName).database('Samples').StormEvents | count
};
foo('help')
```

|计数|
|---|
|59066|

### <a name="use-cluster-inside-functions"></a>在 Functions 内使用 cluster() 

可以重写与上述查询相同的查询，以便在可接收参数 `clusterName` 的函数中使用它 - 该参数会传递到 cluster() 函数中。

```kusto
.create function foo(clusterName:string)
{
    cluster(clusterName).database('Samples').StormEvents | count
};
```

**请注意：** 此类函数只能在本地使用，而不能在跨群集查询中使用。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
