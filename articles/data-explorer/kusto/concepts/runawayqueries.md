---
title: 失控查询 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的失控查询。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: 091fa81fcea3fcbb888d1a2fc93b2d71947eacfe
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105062"
---
# <a name="runaway-queries"></a>失控查询

失控查询是一种[部分查询失败](partialqueryfailures.md)。在查询执行期间超过某个内部[查询限制](querylimits.md)时，就会发生该失败。 

例如，可能会报告以下错误：`HashJoin operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete.`

有几种可能的措施。
* 更改查询以消耗更少的资源。 例如，如果错误指示查询结果集太大，则可以：
  * 通过以下方法限制查询返回的记录数：
     * 使用 [take 运算符](../query/takeoperator.md)
     * 添加其他 [where 子句](../query/whereoperator.md)
  * 通过以下方法减少查询返回的列数：
     * 使用 [project 运算符](../query/projectoperator.md)
     * 使用 [project-away 运算符](../query/projectawayoperator.md)
     * 使用 [project-keep 运算符](../query/project-keep-operator.md)
  * 使用 [summarize 运算符](../query/summarizeoperator.md)获取聚合数据。
* 暂时提高该查询的相关查询限制。 有关详细信息，请参阅[查询限制 - 每个迭代器的内存限制](querylimits.md)。 不过，建议不要使用此方法。 设置这些限制是为了保护群集，并确保单个查询不会中断群集上运行的并发查询。
