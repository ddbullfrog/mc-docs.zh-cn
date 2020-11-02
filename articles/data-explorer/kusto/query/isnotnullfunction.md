---
title: isnotnull() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isnotnull()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: f804821a2139d4f344f6da4ca4b553e4b4db476c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105388"
---
# <a name="isnotnull"></a>isnotnull()

如果参数不为 null，则返回 `true`。

## <a name="syntax"></a>语法

`isnotnull(`[ *value* ]`)`

`notnull(`[ *value* ]`)` - `isnotnull` 的别名

## <a name="example"></a>示例

```kusto
T | where isnotnull(PossiblyNull) | count
```

请注意，还可通过其他方法实现这种效果：

```kusto
T | summarize count(PossiblyNull)
```