---
title: toint() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 toint()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: ac4ca2ebf3e2829650ba2b60637b635caa6b0aaa
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104187"
---
# <a name="toint"></a>toint()

将输入转换为 integer（带符号的 32 位）数字表示形式。

```kusto
toint("123") == int(123)
```

## <a name="syntax"></a>语法

`toint(`Expr`)`

## <a name="arguments"></a>参数

* Expr：将转换为整数的表达式。 

## <a name="returns"></a>返回

如果转换成功，结果将为整数。
如果转换未成功，结果将为 `null`。
 
*注意* ：如果可能，建议使用 [int()](./scalar-data-types/int.md)。