---
title: 批处理 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的批处理。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: d1dddb70d622f63ea50ff39dd51f352cb205b303
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104038"
---
# <a name="batches"></a>批处理

一个查询可以包含多个表格表达式语句，只要以分号 (`;`) 字符分隔它们即可。 这样，该查询会返回多个表格结果。 结果由表格表达式语句产生，并根据查询文本中语句的顺序进行排序。

例如，以下查询产生两个表格结果。 然后，用户代理工具可以显示这些结果以及与每个结果相关联的适当名称（分别为 `Count of events in Florida` 和 `Count of events in Guam`）。

```kusto
StormEvents | where State == "FLORIDA" | count | as ['Count of events in Florida'];
StormEvents | where State == "GUAM" | count | as ['Count of events in Guam']
```

对于多个子查询共享一个通用计算的场景（例如仪表板），批处理很有用。 如果通用计算很复杂，请使用 [materialize() 函数](./materializefunction.md)，并构造查询以使其仅执行一次：

```kusto
let m = materialize(StormEvents | summarize n=count() by State);
m | where n > 2000;
m | where n < 10
```

说明：
* 优先考虑批处理和 [`materialize`](materializefunction.md) 而不是使用 [fork 运算符](forkoperator.md)。
