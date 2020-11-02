---
title: cursor_current()、current_cursor() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 cursor_current()、current_cursor()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/10/2019
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: bc8b83d5e882034238dd48de5edc32cfe47f35d9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104442"
---
# <a name="cursor_current-current_cursor"></a>cursor_current()、current_cursor()

::: zone pivot="azuredataexplorer"

检索范围内数据库游标的当前值。 （名称 `cursor_current` 和 `current_cursor` 是同义词。）

## <a name="syntax"></a>语法

`cursor_current()`

## <a name="returns"></a>返回

返回 `string` 类型的单个值，该值对范围内数据库游标的当前值进行编码。

**说明**

有关数据库游标的其他详细信息，请参阅[数据库游标](../management/databasecursor.md)。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
