---
title: isnotempty() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 isnotempty()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 49d1c9a7030b339972101b05cc7e3bfacd0935ad
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105387"
---
# <a name="isnotempty"></a>isnotempty()

如果参数不为空字符串且不为 null，则返回 `true`。

```kusto
isnotempty("") == false
```

## <a name="syntax"></a>语法

`isnotempty(`[ *value* ]`)`

`notempty(`[ *value* ]`)` - `isnotempty` 的别名

## <a name="example"></a>示例

```kusto
T
| where isnotempty(fieldName)
| count
```
