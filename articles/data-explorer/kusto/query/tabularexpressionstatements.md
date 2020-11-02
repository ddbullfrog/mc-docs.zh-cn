---
title: 表格表达式语句 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的表格表达式语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: f63c5ce177ad9044f84e06d5bf35fde87f8ede79
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104000"
---
# <a name="tabular-expression-statements"></a>表格表达式语句

表格表达式语句是人们在谈论查询时通常会想到的内容。 此语句通常出现在语句列表的最后，其输入和输出都由表或表格数据集组成。

Kusto 对表格表达式语句使用数据流模型。 表格表达式语句的典型结构由这些部分构成：表格数据源（如 Kusto 表）、表格数据运算符（如筛选器和投影），可能还包括呈现运算符  。 这种构成由管道字符（`|`）表示，赋予语句一种非常有规律的形式，直观地表示表格数据从左向右流动。
每个运算符都“从管道”接受一个表格数据集，从运算符正文接受其他输入（包括其他表格数据集），然后向其后的下一个运算符发出表格数据集，如下所示：   

*source1* `|` *operator1* `|` *operator2* `|` *renderInstruction*

在下面的示例中，源是 `Logs`（对当前数据库中表的引用），第一个运算符是 `where`（根据某些因记录而异的谓词，从其输入中筛选出记录），第二个运算符是 `count`（对其输入数据集中的记录数进行计数）：

```kusto
Logs | where Timestamp > ago(1d) | count
```

在下面这个更复杂的示例中，`join` 运算符用于合并来自两个输入数据集的记录：一个是 `Logs` 表的筛选器，另一个是 `Events` 表的筛选器。

```kusto
Logs 
| where Timestamp > ago(1d) 
| join 
(
    Events 
    | where continent == 'Europe'
) on RequestId 
```

## <a name="tabular-data-sources"></a>表格数据源

表格数据源生成记录集，以便由表格数据运算符进一步处理 。 Kusto 支持很多这种源：

* 表引用（指上下文数据库或某种其他群集/数据库中的 Kusto 表。）
* 表格 [range 运算符](rangeoperator.md)。
* [print 运算符](printoperator.md)。
* 返回表的函数的调用。
* [表文本](datatableoperator.md)（“datatable”）。