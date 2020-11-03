---
title: parse_urlquery() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 parse_urlquery()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: d6430a9e012a378d50a0dd2a666cefb9cf0a3e42
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106143"
---
# <a name="parse_urlquery"></a>parse_urlquery()

返回包含查询参数的 `dynamic` 对象。

## <a name="syntax"></a>语法

`parse_urlquery(`*query*`)`

## <a name="arguments"></a>参数

* *查询* ：字符串表示 url 查询。

## <a name="returns"></a>返回

包含查询参数的[动态](./scalar-data-types/dynamic.md)类型的对象。

## <a name="example"></a>示例

```kusto
parse_urlquery("k1=v1&k2=v2&k3=v3")
```

将得到以下结果：

```kusto
 {
    "Query Parameters":"{"k1":"v1", "k2":"v2", "k3":"v3"}",
 }
```

**备注**

* 输入格式应遵循 URL 查询标准 (key=value& ...)
 