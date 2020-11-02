---
title: todynamic()、toobject() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 todynamic()、toobject()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: f09fd9723e8189e61be718457ebf2801394cd686
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105203"
---
# <a name="todynamic-toobject"></a>todynamic(), toobject()

将 `string` 解释为 [JSON 值](https://json.org/)并以 [`dynamic`](./scalar-data-types/dynamic.md) 形式返回值。 

当需要提取 JSON 复合对象的多个元素时，使用它比使用 [extractjson() 函数](./extractjsonfunction.md)更好。

[parse_json()](./parsejsonfunction.md) 函数的别名。

> [!NOTE]
> 尽可能首选使用 [dynamic()](./scalar-data-types/dynamic.md)。

## <a name="syntax"></a>语法

`todynamic(`*json*`)`
`toobject(`*json*`)`

## <a name="arguments"></a>参数

* json：JSON 文档。

## <a name="returns"></a>返回

*json* 指定的 `dynamic` 类型对象。
