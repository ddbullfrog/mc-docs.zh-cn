---
title: isnan() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isnan()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: b2c087f8cc9e01fae9235634375106f2e76520bf
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105393"
---
# <a name="isnan"></a>isnan()

返回输入是否为非数字 (NAN) 值。  

## <a name="syntax"></a>语法

`isnan(`*x*`)`

## <a name="arguments"></a>参数

* x：一个实数。

## <a name="returns"></a>返回

如果 x 是 NaN，则为非零值 (true)，否则为零值 (false)。

## <a name="see-also"></a>请参阅

* 若要检查值是否为 null，请参阅 [isnull()](isnullfunction.md)。
* 若要检查值是否为有限值，请参阅 [isfinite()](isfinitefunction.md)。
* 若要检查值是否为无限值，请参阅 [isinf()](isinffunction.md)。

## <a name="example"></a>示例

```kusto
range x from -1 to 1 step 1
| extend y = (-1*x) 
| extend div = 1.0*x/y
| extend isnan=isnan(div)
```

|x|y|div|isnan|
|---|---|---|---|
|-1|1|-1|0|
|0|0|NaN|1|
|1|-1|-1|0|