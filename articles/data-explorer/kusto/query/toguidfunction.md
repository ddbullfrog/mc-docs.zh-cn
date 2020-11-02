---
title: toguid() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 toguid()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 4d5c00a91d5f7ff9a977ceac9d74aa5cf60d9fcc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104191"
---
# <a name="toguid"></a>toguid()

将输入转换为 [`guid`](./scalar-data-types/guid.md) 表现形式。

```kusto
toguid("70fc66f7-8279-44fc-9092-d364d70fce44") == guid("70fc66f7-8279-44fc-9092-d364d70fce44")
```

> [!NOTE]
> 如果可能，首选使用 [guid()](./scalar-data-types/guid.md)。

## <a name="syntax"></a>语法

`toguid(`Expr`)`

## <a name="arguments"></a>参数

* Expr：将转换为 [`guid`](./scalar-data-types/guid.md) 标量的表达式。 

## <a name="returns"></a>返回

如果转换成功，结果将是 [`guid`](./scalar-data-types/guid.md) 标量。
如果转换未成功，结果将是 `null`。
