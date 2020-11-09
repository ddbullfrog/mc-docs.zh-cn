---
title: Kusto 查询语言中的 where 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 where 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: e32fae40b61b5de82918b4a2cd05e185af46cfc6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104566"
---
# <a name="where-operator"></a>where 运算符

筛选表，获取满足谓词的行子集。

```kusto
T | where fruit=="apple"
```

**Alias** `filter`

## <a name="syntax"></a>语法

*T* `| where` *Predicate*

## <a name="arguments"></a>参数

* *T* ：其记录待筛选的表格输入。
* *谓词* ：针对 T 的列的 `boolean` [表达式](./scalar-data-types/bool.md)。对 T 中的每一行执行此计算。

## <a name="returns"></a>返回

对于 *T* 中的行， *Predicate* 为 `true`。

**注释** Null 值：与 null 值进行比较时，所有筛选函数都返回 false。 可以使用特殊的 null 感知函数编写处理 null 值的查询。

[isnull()](./isnullfunction.md)、[isnotnull()](./isnotnullfunction.md)、[isempty()](./isemptyfunction.md)、[isnotempty()](./isnotemptyfunction.md)。 

**提示**

获取最快性能：

* 在列名称和常量之间 **采用简单比较** 。 （“常量”指基于表的常量 - `now()` 和 `ago()` 均可用，使用 [`let` 语句](./letstatement.md)分配的标量值也可用。）

    例如，使用 `where Timestamp >= ago(1d)` 而非 `where floor(Timestamp, 1d) == ago(1d)`。

* **简单项优先** ：如果有多个子句与 `and` 联合，则将仅涉及一列的子句放在首位。 因此 `Timestamp > ago(1d) and OpId == EventId` 优于其他方法。

有关详细信息，请参阅[可用字符串运算符](./datatypes-string-operators.md)的摘要和[可用数值运算符](./numoperators.md)的摘要。

## <a name="example-simple-comparisons-first"></a>示例：先进行简单比较

```kusto
Traces
| where Timestamp > ago(1h)
    and Source == "MyCluster"
    and ActivityId == SubActivityId 
```

此示例检索那些时间不超过 1 小时、来自“`MyCluster`”源且包含的两个列的值相同的记录。 

请注意，我们将两个列之间的比较放在最后，因为它不能使用索引，因此会强制执行扫描。

## <a name="example-columns-contain-string"></a>示例：列表包含字符串

```kusto
Traces | where * has "Kusto"
```

任何列中出现单词“Kusto”的所有行。
 
