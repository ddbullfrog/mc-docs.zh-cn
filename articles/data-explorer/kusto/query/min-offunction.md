---
title: min_of() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 min_of()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: abf0bee26ba86fc883284d82339681af2a819dea
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103724"
---
# <a name="min_of"></a>min_of()

返回几个计算的数值表达式的最小值。

```kusto
min_of(10, 1, -3, 17) == -3
```

## <a name="syntax"></a>语法

`min_of` `(`*expr_1*`,` *expr_2* ...`)`

## <a name="arguments"></a>参数

* expr_i：要计算的标量表达式。

- 所有参数的类型必须相同。
- 最多支持 64 个参数。

## <a name="returns"></a>返回

所有参数表达式的最小值。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples  -->
```kusto
print result=min_of(10, 1, -3, 17) 
```

|result|
|---|
|-3|
