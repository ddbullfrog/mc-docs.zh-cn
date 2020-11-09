---
title: ingestion_time() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 ingestion_time()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/19/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 823e97aa580f63ec59cf0b5021c4800ba667002b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106336"
---
# <a name="ingestion_time"></a>ingestion_time()

::: zone pivot="azuredataexplorer"

返回当前记录引入的大致时间。

此函数必须在数据引入时为其启用了 [IngestionTime 策略](../management/ingestiontimepolicy.md)的引入数据表的上下文中使用。 否则，此函数会生成 NULL 值。

::: zone-end

::: zone pivot="azuremonitor"

检索引入记录并准备进行查询时的 `datetime`。

::: zone-end

> [!NOTE]
> 此函数返回的值只是近似值，因为引入过程可能需要几分钟才能完成，并且可能会同时进行多个引入活动。 若要以“恰好处理一次”保证方式处理表的所有记录，请使用[数据库游标](../management/databasecursor.md)。

## <a name="syntax"></a>语法

`ingestion_time()`

## <a name="returns"></a>返回

一个 `datetime` 值，该值指定将数据引入到表中的大致时间。

## <a name="example"></a>示例

```kusto
T
| extend ingestionTime = ingestion_time() | top 10 by ingestionTime
```
