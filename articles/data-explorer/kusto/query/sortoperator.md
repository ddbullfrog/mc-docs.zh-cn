---
title: sort 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 sort 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: cdbc0556d642166287e9c873fb3cbc802e252710
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104253"
---
# <a name="sort-operator"></a>sort 运算符 

按照一个或多个列的顺序对输入表的行排序。

```kusto
T | sort by strlen(country) asc, price desc
```

**Alias**

`order`

## <a name="syntax"></a>语法

*T* `| sort by` *expression* [`asc` | `desc`] [`nulls first` | `nulls last`] [`,` ...]

## <a name="arguments"></a>参数

* *T* ：要排序的表输入。
* expression：要作为排序依据的标量表达式。 值的类型必须是数字、日期、时间或字符串。
* `asc` 按升序（即由低到高）排列。 默认值是 `desc`，降序，由高到低。
* `nulls first`（`asc` 顺序的默认值）将把 null 值放在开头，`nulls last`（`desc` 顺序的默认值）将把 null 值放在末尾。

## <a name="example"></a>示例

```kusto
Traces
| where ActivityId == "479671d99b7b"
| sort by Timestamp asc nulls first
```

表 Traces 中具有特定 `ActivityId` 的所有行，按时间戳排序。 如果 `Timestamp` 列包含 null 值，则这些值将显示在结果的前几行。

为了从结果中排除 null 值，请在调用排序运算符之前添加筛选器：

```kusto
Traces
| where ActivityId == "479671d99b7b" and isnotnull(Timestamp)
| sort by Timestamp asc
```