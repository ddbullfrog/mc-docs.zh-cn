---
title: tobool() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 tobool()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: d627aa7338d3aa03016a344f27b38d87817d5a6b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103985"
---
# <a name="tobool"></a>tobool()

将输入转换为布尔值（带符号的 8 位）表示形式。

```kusto
tobool("true") == true
tobool("false") == false
tobool(1) == true
tobool(123) == true
```

## <a name="syntax"></a>语法

`tobool(`*Expr*`)`
`toboolean(`*Expr*`)` (alias)

## <a name="arguments"></a>参数

* Expr：将转换为布尔值的表达式。 

## <a name="returns"></a>返回

如果转换成功，则结果将为布尔值。
如果转换不成功，结果将为 `null`。
