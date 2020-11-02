---
title: isutf8() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isutf8()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: a3a86a93ed885048c3be1cb6c4e532374d4ef003
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105140"
---
# <a name="isutf8"></a>isutf8()

如果参数是有效的 utf8 字符串，则返回 `true`。
    
```kusto
isutf8("some string") == true
```

## <a name="syntax"></a>语法

`isutf8(`[ *value* ]`)`

## <a name="returns"></a>返回

指示参数是否为有效的 utf8 字符串。

## <a name="example"></a>示例

```kusto
T
| where isutf8(fieldName)
| count
```