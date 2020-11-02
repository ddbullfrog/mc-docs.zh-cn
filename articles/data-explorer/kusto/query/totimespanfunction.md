---
title: totimespan() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 totimespan()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 83e3dd2383f083182d285a945a179ee68bf5d827
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105093"
---
# <a name="totimespan"></a>totimespan()

将输入转换为[时间跨度](./scalar-data-types/timespan.md)标量。

```kusto
totimespan("0.00:01:00") == time(1min)
```

## <a name="syntax"></a>语法

`totimespan(Expr)`

## <a name="arguments"></a>参数

* *`Expr`* ：将转换为 [时间跨度](./scalar-data-types/timespan.md)的表达式。

## <a name="returns"></a>返回

如果转换成功，则结果将是[时间跨度](./scalar-data-types/timespan.md)值。
否则，结果将为 null。
