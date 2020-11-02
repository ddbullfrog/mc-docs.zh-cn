---
title: endofday() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 endofday()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 573fd0a765bd864f8e44c965505b47c5090506e0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104660"
---
# <a name="endofday"></a>endofday()

返回包含日期的一天结束的时间，根据偏移量移动（如提供）。

## <a name="syntax"></a>语法

`endofday(`*date* [`,`*offset* ]`)`

## <a name="arguments"></a>参数

* `date`：输入日期。
* `offset`：输入日期（整数，默认值为 0）中的可选偏移天数。

## <a name="returns"></a>返回

表示给定 date 值一天结束的日期/时间（如果指定了偏移量，还包含该信息）。

## <a name="example"></a>示例

```kusto
  range offset from -1 to 1 step 1
 | project dayEnd = endofday(datetime(2017-01-01 10:10:17), offset) 
```

|dayEnd|
|---|
|2016-12-31 23:59:59.9999999|
|2017-01-01 23:59:59.9999999|
|2017-01-02 23:59:59.9999999|