---
title: isnull() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 isnull()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: cbedff06f57f6a7bc88422155855403a5df92882
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105139"
---
# <a name="isnull"></a>isnull()

计算其唯一参数并返回一个 `bool` 值，指示该参数的计算结果是否为 null 值。

```kusto
isnull(parse_json("")) == true
```

## <a name="syntax"></a>语法

`isnull(`Expr`)`

## <a name="returns"></a>返回

true 或 false，具体取决于值是否为 null。

**备注**

* `string` 值不能为 null。 使用 [isempty](./isemptyfunction.md) 可确定类型 `string` 的值是否为空。

|x                |`isnull(x)`|
|-----------------|-----------|
|`""`             |`false`    |
|`"x"`            |`false`    |
|`parse_json("")`  |`true`     |
|`parse_json("[]")`|`false`    |
|`parse_json("{}")`|`false`    |

## <a name="example"></a>示例

```kusto
T | where isnull(PossiblyNull) | count
```