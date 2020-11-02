---
title: 开窗函数 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的开窗函数。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/20/2019
ms.date: 09/30/2020
ms.openlocfilehash: b2dbefa8fe3ab9d7a1e12fed8778175f7a33dff0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104369"
---
# <a name="window-functions-overview"></a>开窗函数概述

开窗函数每次对一个行集中的多个行（记录）进行运算。 与聚合函数不同，开窗函数要求对行集中的行进行序列化（使这些行具有特定的顺序）。 开窗函数可能要依赖于顺序来确定结果。

只能对已序列化的集使用开窗函数。 序列化行集的最简单方法是使用 [serialize 运算符](./serializeoperator.md)。 此运算符以任意方式“冻结”行的顺序。 如果已序列化的行的顺序在语义上很重要，请使用 [sort 运算符](./sortoperator.md)来强制实施某个特定顺序。

与序列化进程所关联的成本并非微不足道。 例如，在许多情形下，它可能会阻止查询并行。 因此，在不必要的情况下，请不要应用序列化。 如有必要，请重新安排查询，以针对尽可能小的行集执行序列化。

## <a name="serialized-row-set"></a>已序列化的行集

可通过以下方法之一序列化任意行集（如表，或表格运算符的输出）：

1. 通过对行集排序。 有关发出已排序行集的运算符的列表，请参阅下文。
2. 通过使用 [serialize 运算符](./serializeoperator.md)。

只要输入已经序列化，许多表格运算符就会序列化输出，即使运算符本身并不确保对结果进行序列化也是如此。 例如，对于 [extend 运算符](./extendoperator.md)、[project 运算符](./projectoperator.md)和 [where 运算符](./whereoperator.md)，这一属性有保证。

## <a name="operators-that-emit-serialized-row-sets-by-sorting"></a>发出已通过排序完成了序列化的行集的运算符

* [order 运算符](./orderoperator.md)
* [sort 运算符](./sortoperator.md)
* [top 运算符](./topoperator.md)
* [top-hitters 运算符](./tophittersoperator.md)
* [top-nested 运算符](./topnestedoperator.md)

## <a name="operators-that-preserve-the-serialized-row-set-property"></a>保留已序列化行集属性的运算符

* [extend 运算符](./extendoperator.md)
* [mv-expand 运算符](./mvexpandoperator.md)
* [parse 运算符](./parseoperator.md)
* [project 运算符](./projectoperator.md)
* [project-away 运算符](./projectawayoperator.md)
* [project-rename 运算符](./projectrenameoperator.md)
* [take 运算符](./takeoperator.md)
* [where 运算符](./whereoperator.md)
