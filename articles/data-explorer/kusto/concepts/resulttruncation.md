---
title: Kusto 查询结果集超过了内部限制 - Azure 数据资源管理器
description: 本文介绍“查询结果集已超过 Azure 数据资源管理器中的内部限制”。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/23/2018
ms.date: 10/29/2020
ms.openlocfilehash: 4db77e545120330a7426094de679517e07f60db7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103553"
---
# <a name="query-result-set-has-exceeded-the-internal--limit"></a>查询结果集已超过内部限制

“查询结果集已超过内部限制”是一种[部分查询失败](partialqueryfailures.md)，当查询结果超出了以下两个限制之一时发生：
* 记录数的限制（`record count limit`，默认设置为 500,000）
* 总数据量的限制（`data size limit`，默认设置为 67,108,864 (64MB)）

有几种可能的措施：

* 更改查询以消耗更少的资源。 
  例如，可以：
  * 使用 [take 运算符](../query/takeoperator.md)或添加其他 [where 子句](../query/whereoperator.md)来限制查询返回的记录数
  * 尝试减少查询返回的列数。 使用 [project 运算符](../query/projectoperator.md)、[project-away 运算符](../query/projectawayoperator.md)或 [project-keep 运算符](../query/project-keep-operator.md)
  * 使用 [summarize 运算符](../query/summarizeoperator.md)获取聚合数据
* 暂时提高该查询的相关查询限制。 有关详细信息，请参阅[查询限制](querylimits.md)下的“结果截断”

 > [!NOTE] 
 > 建议你不要提高查询限制，因为设置这些限制是为了保护群集。 限制可确保单个查询不会中断在群集上运行的并发查询。
  
