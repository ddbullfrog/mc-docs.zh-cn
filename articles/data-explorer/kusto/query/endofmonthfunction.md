---
title: endofmonth() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 endofmonth()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: a8d28e6e1c6d9237594826cf5a452b256af476ba
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104658"
---
# <a name="endofmonth"></a>endofmonth()

返回包含日期的一月的终点，根据偏移量移动（如提供）。

## <a name="syntax"></a>语法

`endofmonth(`*date* [`,`*offset* ]`)`

## <a name="arguments"></a>参数

* `date`：输入日期。
* `offset`：输入日期中的可选偏移月数（整数，默认值为 0）。

## <a name="returns"></a>返回

一个日期/时间，表示给定 date 值的那一月的结束（如果指定了偏移量，则还包含该信息）。

## <a name="example"></a>示例

```kusto
  range offset from -1 to 1 step 1
 | project monthEnd = endofmonth(datetime(2017-01-01 10:10:17), offset) 
```

|monthEnd|
|---|
|2016-12-31 23:59:59.9999999|
|2017-01-31 23:59:59.9999999|
|2017-02-28 23:59:59.9999999|