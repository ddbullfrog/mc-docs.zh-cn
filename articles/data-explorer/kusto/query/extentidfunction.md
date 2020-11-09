---
title: extent_id() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 extent_id()。
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
ms.openlocfilehash: ab4eb1870a96c045dc968452e6c1a3f61a6074e8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105154"
---
# <a name="extent_id"></a>extent_id()

::: zone pivot="azuredataexplorer"

返回标识当前记录所在的数据分片（“盘区”）的唯一标识符。

将此函数应用于未附加到数据分片的计算数据将返回空的 guid（全为零）。

## <a name="syntax"></a>语法

`extent_id()`

## <a name="returns"></a>返回

`guid` 类型的值，该值标识当前记录的数据分片或空 guid（全为零）。

## <a name="example"></a>示例

下面的示例演示如何获取一个列表，其中包含其记录是一小时前的所有数据分片，这些记录具有列 `ActivityId` 的特定值。 它表明，一些查询运算符（这里是 `where` 运算符，以及 `extend` 和 `project`）保留了有关承载记录的数据分片的信息。

```kusto
T
| where Timestamp > ago(1h)
| where ActivityId == 'dd0595d4-183e-494e-b88e-54c52fe90e5a'
| extend eid=extent_id()
| summarize by eid
```

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
