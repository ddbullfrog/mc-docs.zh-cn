---
title: array_length() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 array_length()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 7ae2ad9bdaf76cb2c1fca1ec6aedc718e467ff82
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106169"
---
# <a name="array_length"></a>array_length()

计算动态数组中的元素数。

## <a name="syntax"></a>语法

`array_length(`array`)`

## <a name="arguments"></a>参数

* array：一个 `dynamic` 值。

## <a name="returns"></a>返回

*array* 中的元素数；如果 *array* 不是数组，则返回`null`。

## <a name="examples"></a>示例

```kusto
print array_length(parse_json('[1, 2, 3, "four"]')) == 4

print array_length(parse_json('[8]')) == 1

print array_length(parse_json('[{}]')) == 1

print array_length(parse_json('[]')) == 0

print array_length(parse_json('{}')) == null

print array_length(parse_json('21')) == null
```