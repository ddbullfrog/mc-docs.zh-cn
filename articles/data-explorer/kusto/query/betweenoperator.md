---
title: between 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 between 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: 925cebe415b888d9f2c888e12c6e5b774a08c38f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103737"
---
# <a name="between-operator"></a>between 运算符

匹配包含范围内的输入。

```kusto
Table1 | where Num1 between (1 .. 10)
Table1 | where Time between (datetime(2017-01-01) .. datetime(2017-01-01))
```

`between` 可以对任何数值、日期时间或时间跨度表达式执行运算。
 
## <a name="syntax"></a>语法

*T* `|` `where` *expr* `between` `(`*leftRange*` .. `*rightRange*`)`   
 
如果 expr 表达式为时间跨度，则提供另一个糖衣语法：

*T* `|` `where` *expr* `between` `(`*leftRangeDateTime*` .. `*rightRangeTimespan*`)`   

## <a name="arguments"></a>参数

* *T* - 待匹配记录的表格输入。
* *expr* - 要筛选的表达式。
* *leftRange* - 左侧范围（含）的表达式。
* *rightRange* - 右侧范围（含）的表达式。

## <a name="returns"></a>返回

T 中谓词（expr  >=  leftRange 和 expr  <=  rightRange）的行的计算结果为 `true`    。

## <a name="examples"></a>示例  

**使用“between”运算符筛选数值**  

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 100 step 1
| where x between (50 .. 55)
```

|x|
|---|
|50|
|51|
|52|
|53|
|54|
|55|

**使用“between”运算符筛选日期时间**  

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. datetime(2007-07-30))
| count 
```

|计数|
|---|
|476|

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. 3d)
| count 
```

|计数|
|---|
|476|
