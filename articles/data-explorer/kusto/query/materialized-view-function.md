---
title: materialized_view()（范围函数）- Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 materialized_view() 函数。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: 36cb4f69e50d80b343cdb9a2496410e59fbe4109
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106752"
---
# <a name="materialized_view-function"></a>materialized_view() 函数

引用[具体化视图](../management/materialized-views/materialized-view-overview.md)的具体化部分。 

`materialized_view()` 函数支持仅查询视图的具体化部分，同时指定用户愿容忍的最大延迟。 此选项不保证返回最新记录，但与查询整个视图相比，此选项始终更加高效。 此函数适用于愿舍弃一些新鲜度以提高性能的方案，例如在遥测仪表板中。

<!--- csl --->
```
materialized_view('ViewName')
```

## <a name="syntax"></a>语法

`materialized_view(`ViewName`,` [max_age]`)` 

## <a name="arguments"></a>参数

* ViewName：`materialized view` 的名称。
* max_age：可选。 如果未提供，则仅返回视图的具体化部分。 如果已提供，并且上次具体化时间晚于 [@now -  max_age]，则函数将返回视图的具体化部分。 否则，返回整个视图（等同于直接查询 ViewName。 

## <a name="examples"></a>示例

仅查询视图的具体化部分，与视图上次具体化的时间无关。

<!-- csl -->
```
materialized_view("ViewName")
```

仅在最近 10 分钟内具体化了视图时才查询具体化部分。 如果具体化部分的时间在 10 分钟之前，则返回完整视图。 预计此选项比查询具体化部分低效。

<!-- csl -->
```
materialized_view("ViewName", 10m)
```

## <a name="notes"></a>说明

* 创建视图后，可以以查询数据库中的其他任何表的方式对其进行查询，包括参与跨群集/跨数据库查询。
* 通配符并集或搜索中不包含具体化视图。
* 查询视图的语法是视图名称（如表格引用）。
* 查询具体化视图将始终根据引入源表的所有记录返回最新结果。 该查询会将视图的具体化部分与源表中的所有未具体化记录结合起来。 有关详细信息，请参阅[幕后](../management/materialized-views/materialized-view-overview.md#how-materialized-views-work)。
