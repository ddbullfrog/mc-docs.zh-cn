---
title: isinf() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 isinf()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 4d7dfd163478a6c6bcb524ccb3f5eed9c81bafce
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104029"
---
# <a name="isinf"></a>isinf()

返回输入是否为无限值（正值或负值）。  

## <a name="syntax"></a>语法

`isinf(`*x*`)`

## <a name="arguments"></a>参数

* x：一个实数。

## <a name="returns"></a>返回

如果 x 为正无穷或负无穷，则为非零值 (true)，否则为零值 (false)。

## <a name="see-also"></a>请参阅

* 若要检查值是否为 null，请参阅 [isnull()](isnullfunction.md)。
* 若要检查值是否有限，请参阅 [isfinite()](isfinitefunction.md)。
* 若要检查值是否为 NAN（非数字），请参阅 [isnan()](isnanfunction.md)。

## <a name="example"></a>示例

```kusto
range x from -1 to 1 step 1
| extend y = 0.0
| extend div = 1.0*x/y
| extend isinf=isinf(div)
```

|x|y|div|isinf|
|---|---|---|---|
|-1|0|-∞|1|
|0|0|NaN|0|
|1|0|∞|1|
