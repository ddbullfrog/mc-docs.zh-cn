---
title: binary_all_or()（聚合函数）- Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 binary_all_or()（聚合函数）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/24/2020
ms.date: 10/29/2020
ms.openlocfilehash: 7c145bc677326e15a3616b2861746b82caf5d879
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106150"
---
# <a name="binary_all_or-aggregation-function"></a>binary_all_or()（聚合函数）

使用每个汇总组的二进制 `OR` 操作累计值（如果没有分组就完成了汇总，则返回总计值）。

* 只能在 [summarize](summarizeoperator.md) 内的聚合上下文中使用

## <a name="syntax"></a>语法

summarize `binary_all_or(`*Expr*`)`

## <a name="arguments"></a>参数

* *Expr* ：长整型数字。

## <a name="returns"></a>返回

返回使用二进制 `OR` 操作对每个汇总组的记录进行聚合所得的值（如果没有分组就完成了汇总，则返回总计值）。

## <a name="example"></a>示例

使用二进制 `OR` 操作生成“cafe-food”：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(num:long)
[
  0x88888008,
  0x42000000,
  0x00767000,
  0x00000005, 
]
| summarize result = toupper(tohex(binary_all_or(num)))
```

|result|
|---|
|CAFEF00D|
