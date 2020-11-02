---
title: 跨群集联接 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的跨群集联接。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 481d821c5edc42eb17b496585856e63d2a304b21
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105596"
---
# <a name="cross-cluster-join"></a>跨群集联接

::: zone pivot="azuredataexplorer"

有关跨群集查询的常规讨论，请参阅[跨群集查询或跨数据库查询](cross-cluster-or-database-queries.md)

可以对位于不同群集上的数据集执行联接操作。 例如：

```kusto
T | ... | join (cluster("SomeCluster").database("SomeDB").T2 | ...) on Col1 // (1)

cluster("SomeCluster").database("SomeDB").T | ... | join (cluster("SomeCluster2").database("SomeDB2").T2 | ...) on Col1 // (2)
```

在上面的示例中，假设当前群集不是“SomeCluster”或“SomeCluster2”，则联接操作是跨群集联接。

在以下示例中：

```kusto
cluster("SomeCluster").database("SomeDB").T | ... | join (cluster("SomeCluster").database("SomeDB2").T2 | ...) on Col1 
```

联接操作不是跨群集联接，因为它的两个操作数都源自同一群集。

当 Kusto 遇到跨群集联接时，它将自动决定执行联接操作的位置。 此决定会出现以下三个可能的结果之一：

* 在左操作数的群集上执行联接操作，该群集将首先提取右操作数。 （示例 (1) 中的联接将在本地群集上执行）
* 在右操作数的群集上执行联接操作，该群集将首先提取左操作数。 （示例 (2) 中的联接将在“SomeCluster2”上执行）
* 在本地执行联接操作（即在接收到查询的群集上执行），本地群集将首先提取这两个操作数。

实际决定取决于特定的查询。 自动联接远程处理策略为（简化版本）：“如果其中一个操作数是本地操作，则联接将在本地执行。 如果两个操作数都是远程操作，则联接将在右操作数的群集上执行”。

有时，如果不遵循自动远程处理策略，查询的性能可得到提高。 在这种情况下，在最大操作数的群集上执行联接操作。

如果在示例 (1) 中，`T | ...` 生成的数据集比 `cluster("SomeCluster").database("SomeDB").T2 | ...` 生成的数据集小得多，则在“SomeCluster”上执行联接操作会更高效。

可以通过提供 Kusto 联接远程处理提示来完成此操作。 语法为：

```kusto
T | ... | join hint.remote=<strategy> (cluster("SomeCluster").database("SomeDB").T2 | ...) on Col1
```

下面是 `strategy` 的合法值
* `left` - 在左操作数的群集上执行联接 
* `right` - 在右操作数的群集上执行联接
* `local` - 在当前群集的群集上执行联接
* `auto` -（默认）让 Kusto 进行自动远程处理决策

> [!Note]
> 如果提示的策略不适用于联接操作，则 Kusto 将忽略联接远程处理提示。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
