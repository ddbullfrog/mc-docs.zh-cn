---
title: project-away 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 project-away 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 8765822af7e0f6ef3ae8733502545113e7075034
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104196"
---
# <a name="project-away-operator"></a>project-away 运算符

从输入中选择要从输出中排除的列。

```kusto
T | project-away price, quantity, zz*
```

结果中列的顺序取决于列在表中的原始顺序。 仅删除已指定为参数的列。 其他列会包括在结果中。 （另请参阅 `project`。）

## <a name="syntax"></a>语法

*T* `| project-away` *ColumnNameOrPattern* [`,` ...]

## <a name="arguments"></a>参数

* *T* ：输入表
* ColumnNameOrPattern：要从输出中删除的列或列通配符模式的名称。

## <a name="returns"></a>返回

一个包含未指定为参数的列的表。 包含与输入表相同的行数。

> [!TIP]
>
> * 若要重命名列，请使用 [`project-rename`](projectrenameoperator.md)。
> * 若要对列重新排序，请使用 [`project-reorder`](projectreorderoperator.md)。
> * 可以 `project-away` 存在于原始表中或已作为查询的一部分进行计算的任何列。

## <a name="examples"></a>示例

输入表 `T` 具有属于 `long` 类型的三列：`A`、`B` 和 `C`。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(A:long, B:long, C:long) [1, 2, 3]
| project-away C    // Removes column C from the output
```

|A|B|
|---|---|
|1|2|

删除以“a”开头的列。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print a2='a2', b = 'b', a3='a3', a1='a1'
| project-away a*
```

|b|
|---|
|b|

## <a name="see-also"></a>请参阅

若要从输入中选择要保留在输出中的列，请使用 [project-keep](project-keep-operator.md)。
