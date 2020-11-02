---
title: getschema 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 getschema 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 366588a045c512cf06527ea434f052cf9351d5dd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103889"
---
# <a name="getschema-operator"></a>getschema 运算符 

生成表示输入的表格架构的表。

```kusto
T | summarize MyCount=count() by Country | getschema 
```

## <a name="syntax"></a>语法

T `| ` `getschema`

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| top 10 by Timestamp
| getschema
```

|ColumnName|ColumnOrdinal|数据类型|ColumnType|
|---|---|---|---|
|Timestamp|0|System.DateTime|datetime|
|语言|1|System.String|string|
|页面|2|System.String|string|
|视图|3|System.Int64|long
|BytesDelivered|4|System.Int64|long
