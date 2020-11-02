---
title: startofday() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 startofday()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 02089f398173ae87d0c24441875cc0a18364ddd7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103657"
---
# <a name="startofday"></a>startofday()

返回包含日期的一天的起点，根据偏移量移动（如提供）。

## <a name="syntax"></a>语法

`startofday(`*date* [`,`*offset* ]`)`

## <a name="arguments"></a>参数

* `date`：输入日期。
* `offset`：输入日期（整数，默认值为 0）中的可选偏移天数。 

## <a name="returns"></a>返回

一个日期/时间，表示给定 date 值的那一天的开始（如果指定了偏移量，则还包含该信息）。

## <a name="example"></a>示例

```kusto
  range offset from -1 to 1 step 1
 | project dayStart = startofday(datetime(2017-01-01 10:10:17), offset) 
```

|dayStart|
|---|
|2016-12-31 00:00:00.0000000|
|2017-01-01 00:00:00.0000000|
|2017-01-02 00:00:00.0000000|