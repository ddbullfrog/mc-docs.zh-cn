---
title: Startofmonth() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 startofmonth()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 00d773f70c1d5deb42f0a68c9b516af8f2753ba6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105105"
---
# <a name="startofmonth"></a>startofmonth()

返回包含日期的一月的起点，根据偏移量移动（如提供）。

## <a name="syntax"></a>语法

`startofmonth(`*date* [`,`*offset* ]`)`

## <a name="arguments"></a>参数

* `date`：输入日期。
* `offset`：输入日期中的可选偏移月数（整数，默认值为 0）。

## <a name="returns"></a>返回

表示给定“日期”值的月份开始的日期/时间（带偏移量，如果指定了的话）。

## <a name="example"></a>示例

```kusto
  range offset from -1 to 1 step 1
 | project monthStart = startofmonth(datetime(2017-01-01 10:10:17), offset) 
```

|monthStart|
|---|
|2016-12-01 00:00:00.0000000|
|2017-01-01 00:00:00.0000000|
|2017-02-01 00:00:00.0000000|