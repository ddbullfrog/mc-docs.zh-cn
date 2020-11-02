---
title: not() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 not()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4c7e371f9242cafc484d1320a222fdebb1067e07
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103857"
---
# <a name="not"></a>not()

对其 `bool` 参数的值取反。

```kusto
not(false) == true
```

## <a name="syntax"></a>语法

`not(`expr`)`

## <a name="arguments"></a>参数

* expr：要取反的 `bool` 表达式。

## <a name="returns"></a>返回

返回其 `bool` 参数的反向逻辑值。