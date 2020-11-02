---
title: toscalar() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 toscalar()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 09/30/2020
ms.openlocfilehash: aff4ead435c3215e7f5081caef52056dead774f3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105350"
---
# <a name="toscalar"></a>toscalar()

返回所计算的表达式的标量常数值。 

此函数对于需要暂存计算的查询很有用。 例如，计算事件总数，然后使用结果筛选超过所有事件的特定百分比的组。

## <a name="syntax"></a>语法

`toscalar(`*Expression*`)`

## <a name="arguments"></a>参数

* *表达式* ：将针对标量转换进行计算的表达式。

## <a name="returns"></a>返回

所计算表达式的标量常数值。
如果结果是表格格式，则会采用第一列和第一行进行转换。

> [!TIP]
> 使用 `toscalar()` 时，可以使用 [let 语句](letstatement.md)以提高查询的可读性。

**备注**

可以在查询执行期间对 `toscalar()` 计算常数次数。
`toscalar()` 函数不能应用于行级（for-each-row 方案）。

## <a name="examples"></a>示例

将 `Start`、`End` 和 `Step` 作为标量常数计算，并将结果用于 `range` 计算。

```kusto
let Start = toscalar(print x=1);
let End = toscalar(range x from 1 to 9 step 1 | count);
let Step = toscalar(2);
range z from Start to End step Step | extend start=Start, end=End, step=Step
```

|z|start|end|步骤|
|---|---|---|---|
|1|1|9|2|
|3|1|9|2|
|5|1|9|2|
|7|1|9|2|
|9|1|9|2|
