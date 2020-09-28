---
title: isnotempty() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 isnotempty()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/24/2020
ms.openlocfilehash: abb448628795772f008dbf3fc03cad9d9f70ff54
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146760"
---
# <a name="isnotempty"></a>isnotempty()

如果参数不为空字符串且不为 null，则返回 `true`。

```kusto
isnotempty("") == false
```

## <a name="syntax"></a>语法

`isnotempty(`[*value*]`)`

`notempty(`[*value*]`)` - `isnotempty` 的别名

## <a name="example"></a>示例

```kusto
T
| where isnotempty(fieldName)
| count
```
