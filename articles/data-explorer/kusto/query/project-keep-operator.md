---
title: project-keep 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 project-keep 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 10/21/2020
ms.date: 10/30/2020
ms.openlocfilehash: bac58ab21e0c0957d32f6236300e60dc37c5b5c9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106544"
---
# <a name="project-keep-operator"></a>project-keep 运算符

从输入中选择要保留在输出中的列。

```kusto
T | project-keep price, quantity, zz*
```

结果中列的顺序取决于列在表中的原始顺序。 仅保留已指定为参数的列。 将从结果中排除其他列。 另请参阅 [`project`](projectoperator.md)。

## <a name="syntax"></a>语法

*T* `| project-keep` *ColumnNameOrPattern* [`,` ...]

## <a name="arguments"></a>参数

* *T* ：输入表
* ColumnNameOrPattern：要保留在输出中的列或列通配符模式的名称。

## <a name="returns"></a>返回

一个包含指定为参数的列的表。 包含与输入表相同的行数。

> [!TIP]
>* 若要重命名列，请使用 [`project-rename`](projectrenameoperator.md)。
>* 若要对列重新排序，请使用 [`project-reorder`](projectreorderoperator.md)。
>* 可以 `project-keep` 存在于原始表中或已作为查询的一部分进行计算的任何列。

## <a name="example"></a>示例

输入表 `T` 具有属于 `long` 类型的三列：`A`、`B` 和 `C`。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(A1:long, A2:long, B:long) [1, 2, 3]
| project-keep A*    // Keeps only columns A1 and A2 in the output
```

|A1|A2|
|---|---|
|1|2|

## <a name="see-also"></a>请参阅

若要从输入中选择要从输出中排除的列，请使用 [project-away](projectawayoperator.md)。
