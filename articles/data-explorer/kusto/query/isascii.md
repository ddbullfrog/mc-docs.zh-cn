---
title: isascii() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isascii()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 2e99fe5f2e3dca6c5fc6983621f9a7597b4c92dd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104481"
---
# <a name="isascii"></a>isascii()

如果参数是有效的 ascii 字符串，则返回 `true`。
    
```kusto
isascii("some string") == true
```

## <a name="syntax"></a>语法

`isascii(`[ *value* ]`)`

## <a name="returns"></a>返回

指示参数是否为有效的 ascii 字符串。

## <a name="example"></a>示例

```kusto
T
| where isascii(fieldName)
| count
```