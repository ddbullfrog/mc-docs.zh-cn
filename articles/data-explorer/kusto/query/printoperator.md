---
title: print 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 print 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/16/2019
ms.date: 10/29/2020
ms.openlocfilehash: 2acf1e3dd294a8930c3d76678ca2f43f9b7dfc47
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104198"
---
# <a name="print-operator"></a>print 运算符

输出单个行，其中包含一个或多个标量表达式。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print x=1, s=strcat("Hello", ", ", "World!")
```

## <a name="syntax"></a>语法

`print` [ *ColumnName* `=`] *ScalarExpression* [',' ...]

## <a name="arguments"></a>参数

* ColumnName：要分配给输出的单数列的选项名称。
* ScalarExpression：要计算的标量表达式。

## <a name="returns"></a>返回

一个单列单行表，其单个单元格的值为已计算的 ScalarExpression。

## <a name="examples"></a>示例

`print` 运算符非常有用，可以快速计算一个或多个标量表达式，并用生成的值创建一个单行表。
例如：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print 0 + 1 + 2 + 3 + 4 + 5, x = "Wow!"
```
<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print banner=strcat("Hello", ", ", "World!")
```
