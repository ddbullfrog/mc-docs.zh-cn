---
title: array_iif() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 array_iif()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 04/28/2019
ms.date: 10/29/2020
ms.openlocfilehash: c1f4117252723d342cfd76f6338fe701b25a3724
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106171"
---
# <a name="array_iif"></a>array_iif()

动态数组上元素对应的 iif 函数。

另一个别名：array_iff()。

## <a name="syntax"></a>语法

`array_iif(`ConditionArray, IfTrue, IfFalse]`)`  

## <a name="arguments"></a>参数

* conditionArray：布尔值或数值组成的输入数组，必须是动态数组。
* ifTrue：值或基元值（ConditionArray 的相应值为 true 时的结果值）组成的输入数组 。
* ifFalse：值或基元值（ConditionArray 的相应值为 false 时的结果值）组成的输入数组 。

**说明**

* 结果长度是 conditionArray 的长度。
* 数字条件值被视为 condition != 0 。
* 非数字/NULL 条件值在结果的相应索引中将为 NULL。
* 缺失值（长度较短的数组）被视为 NULL。

## <a name="returns"></a>返回

根据条件数组的相应值，从 IfTrue 或 IfFalse [array] 值中获取的值组成的动态数组 。

## <a name="example"></a>示例

```kusto
print condition=dynamic([true,false,true]), l=dynamic([1,2,3]), r=dynamic([4,5,6]) 
| extend res=array_iif(condition, l, r)
```

|condition|l|r|res|
|---|---|---|---|
|[true, false, true]|[1, 2, 3]|[4, 5, 6]|[1, 5, 3]|
