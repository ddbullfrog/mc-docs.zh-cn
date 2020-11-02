---
title: max_of() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 max_of()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1b1d5706c93c81309986b99f14e1b0b2db950b0a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104891"
---
# <a name="max_of"></a>max_of()

返回几个计算的数值表达式的最大值。

```kusto
max_of(10, 1, -3, 17) == 17
```

## <a name="syntax"></a>语法

`max_of` `(`*expr_1*`,` *expr_2* ...`)`

## <a name="arguments"></a>参数

* expr_i：要计算的标量表达式。

- 所有参数的类型必须相同。
- 最多支持 64 个参数。

## <a name="returns"></a>返回

所有参数表达式的最大值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples  -->
```kusto
print result = max_of(10, 1, -3, 17) 
```

|result|
|---|
|17|
