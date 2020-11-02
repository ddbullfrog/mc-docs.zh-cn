---
title: endofyear() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 endofyear()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: bf0798c835e025d6a10e5af0fc65c295e7e22cd6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103939"
---
# <a name="endofyear"></a>endofyear()

返回包含日期的一年的终点，根据偏移量移动（如提供）。

## <a name="syntax"></a>语法

`endofyear(`*date* [`,`*offset* ]`)`

## <a name="arguments"></a>参数

* `date`：输入日期。
* `offset`：输入日期（整数，默认值为 0）中的可选偏移年数。

## <a name="returns"></a>返回

表示给定“日期”值的年份结束的日期/时间（带偏移量，如果指定了的话）。

## <a name="example"></a>示例

```kusto
  range offset from -1 to 1 step 1
 | project yearEnd = endofyear(datetime(2017-01-01 10:10:17), offset) 
```

|yearEnd|
|---|
|2016-12-31 23:59:59.9999999|
|2017-12-31 23:59:59.9999999|
|2018-12-31 23:59:59.9999999|