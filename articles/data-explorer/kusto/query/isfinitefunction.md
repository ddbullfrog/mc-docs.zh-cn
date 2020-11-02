---
title: isfinite() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isfinite()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 658d1d3d7123d0f7122d50a50b90737112328bb2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104030"
---
# <a name="isfinite"></a>isfinite()

返回输入是否为有限值（既不是无穷值也不是 NAN）。

## <a name="syntax"></a>语法

`isfinite(`*x*`)`

## <a name="arguments"></a>参数

* x：一个实数。

## <a name="returns"></a>返回

如果 x 是有限值，则为非零值 (true)，否则为零值 (false)。

## <a name="see-also"></a>请参阅

* 若要检查值是否为 null，请参阅 [isnull()](isnullfunction.md)。
* 若要检查值是否为无限值，请参阅 [isinf()](isinffunction.md)。
* 若要检查值是否为 NAN（非数字），请参阅 [isnan()](isnanfunction.md)。

## <a name="example"></a>示例

```kusto
range x from -1 to 1 step 1
| extend y = 0.0
| extend div = 1.0*x/y
| extend isfinite=isfinite(div)
```

|x|y|div|isfinite|
|---|---|---|---|
|-1|0|-∞|0|
|0|0|NaN|0|
|1|0|∞|0|