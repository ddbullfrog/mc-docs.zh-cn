---
title: parse_url() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 parse_url()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4fa43bd26b01d7fc46e1d82ffdf97560127af4d8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106144"
---
# <a name="parse_url"></a>parse_url()

分析绝对 URL `string`，并返回包含 URL 部分的 `dynamic` 对象。


## <a name="syntax"></a>语法

`parse_url(`url`)`

## <a name="arguments"></a>参数

* url：一个字符串，表示 URL 或 URL 的查询部分。

## <a name="returns"></a>返回

一个[动态](./scalar-data-types/dynamic.md)类型对象，其中包含以下 URL 组件：Scheme、Host、Port、Path、Username、Password、Query Parameters、Fragment。

## <a name="example"></a>示例

```kusto
T | extend Result = parse_url("scheme://username:password@host:1234/this/is/a/path?k1=v1&k2=v2#fragment")
```

将得到以下结果：

```
 {
    "Scheme":"scheme",
    "Host":"host",
    "Port":"1234",
    "Path":"this/is/a/path",
    "Username":"username",
    "Password":"password",
    "Query Parameters":"{"k1":"v1", "k2":"v2"}",
    "Fragment":"fragment"
 }
```