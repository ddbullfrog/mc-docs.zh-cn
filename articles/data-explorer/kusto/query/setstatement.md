---
title: Set 语句 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 Set 语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: dce9ac6e148384aa8303355c7e2e865467d378e2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106257"
---
# <a name="set-statement"></a>Set 语句

::: zone pivot="azuredataexplorer"

`set` 语句用于设置查询持续时间的查询选项。
查询选项控制查询的执行方式并返回结果。 它们可以是布尔标志（默认关闭），也可以是整数值。 一个查询可以包含零个、一个或多个 Set 语句。 Set 语句仅影响按程序顺序尾随这些语句的表格表达式语句。

* 还可以通过在 `ClientRequestProperties` 对象中设置查询选项以编程方式启用查询选项。 [请参阅](../api/netfx/request-properties.md)。
  
* 查询选项并不是 Kusto 语言的正式组成部分，可以修改它，且不会视为中断语言的更改。

## <a name="syntax"></a>语法

`set` *OptionName* [`=` *OptionValue* ]

## <a name="example"></a>示例

```kusto
set querytrace;
Events | take 100
```

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
