---
title: project-reorder 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 project-reorder 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: deb9b242f264465a596c8140580ca3d353be9c54
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105029"
---
# <a name="project-reorder-operator"></a>project-reorder 运算符

对结果输出中的列重新排序。

```kusto
T | project-reorder Col2, Col1, Col* asc
```

## <a name="syntax"></a>语法

T `| project-reorder` ColumnNameOrPattern [`asc`|`desc`] [`,` ...] 

## <a name="arguments"></a>参数

* *T* ：输入表。
* ColumnNameOrPattern：添加到输出中的列或列通配符模式的名称。
* 对于通配符模式：指定 `asc` 或 `desc` 使用其名称按升序或降序对列进行排序。 如果未指定 `asc` 或 `desc`，则顺序取决于源表中显示的匹配列。

> [!NOTE]
> * 在 ColumnNameOrPattern 模糊匹配中，该列出现在与模式匹配的第一个位置。
> * 为 `project-reorder` 指定列是可选操作。 未显式指定的列将显示为输出表的最后一列。
> * 若要删除列，请使用 [`project-away`](projectawayoperator.md)。
> * 若要选择要保留的列，请使用 [`project-keep`](project-keep-operator.md)。
> * 若要重命名列，请使用 [`project-rename`](projectrenameoperator.md)。

## <a name="returns"></a>返回

一张表，其中包含按运算符参数指定的顺序排列的列。 `project-reorder` 不会重命名或删除表中的列，因此，源表中存在的所有列都将出现在结果表中。

## <a name="examples"></a>示例

对包含三列 (a, b, c) 的表重新排序，使第二列 (b) 显示在最前面。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print a='a', b='b', c='c'
|  project-reorder b
```

|b|a|c|
|---|---|---|
|b|a|c|

对表中的列重新排序，使以 `a` 开头的列显示在其他列之前。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print b = 'b', a2='a2', a3='a3', a1='a1'
|  project-reorder a* asc
```

|a1|a2|a3|b|
|---|---|---|---|
|a1|a2|a3|b|
