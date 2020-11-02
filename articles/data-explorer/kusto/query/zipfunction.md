---
title: zip() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 zip()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 0ccac041365cb982ea5e293b8b24e4bb9c1dfa69
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104367"
---
# <a name="zip"></a>zip()

`zip` 函数接受任意数量的 `dynamic` 数组，并返回一个数组，其元素都是一个数组，后者包含同一索引的输入数组的元素。

## <a name="syntax"></a>语法

`zip(`*array1*`,` *array2*`, ... )`

## <a name="arguments"></a>参数

2 到 16 个动态数组。

## <a name="examples"></a>示例

以下示例返回 `[[1,2],[3,4],[5,6]]`：

```kusto
print zip(dynamic([1,3,5]), dynamic([2,4,6]))
```

以下示例返回 `[["A",{}], [1,"B"], [1.5, null]]`：

```kusto
print zip(dynamic(["A", 1, 1.5]), dynamic([{}, "B"]))
```

以下示例返回 `[[1,"one"],[2,"two"],[3,"three"]]`：

```kusto
datatable(a:int, b:string) [1,"one",2,"two",3,"three"]
| summarize a = make_list(a), b = make_list(b)
| project zip(a, b)
```