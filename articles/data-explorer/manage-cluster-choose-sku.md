---
title: 为 Azure 数据资源管理器群集选择正确的计算 SKU
description: 本文介绍如何为 Azure 数据资源管理器群集选择最佳计算 SKU 大小。
author: orspod
ms.author: v-tawe
ms.reviewer: avnera
ms.service: data-explorer
ms.topic: how-to
origin.date: 07/14/2019
ms.date: 09/30/2020
ms.openlocfilehash: 30f83b4d74f677fbc1ea0f909def8714520e4567
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105693"
---
# <a name="select-the-correct-compute-sku-for-your-azure-data-explorer-cluster"></a>为 Azure 数据资源管理器群集选择正确的计算 SKU 

为某个不断变化的工作负荷创建新群集或优化群集时，Azure 数据资源管理器会提供多个虚拟机 (VM) SKU 供你选择。 这些计算 SKU 经过精选，可为任何工作负载提供最佳性价比。 

数据管理群集的大小和 VM SKU 完全由 Azure 数据资源管理器服务进行管理。 它们由引擎的 VM 大小和引入工作负荷等因素决定。 

随时可通过[纵向扩展群集](manage-cluster-vertical-scaling.md)来更改引擎群集的计算 SKU。 最好是从适合初始方案的最小 SKU 大小开始。 请注意，使用新的 SKU 重新创建群集时，纵向扩展群集会导致最长 30 分钟的停机。

> [!TIP]
> [计算预留实例 (RI)](https://docs.microsoft.com/azure/virtual-machines/windows/prepay-reserved-vm-instances) 适用于 Azure 数据资源管理器群集。  

本文介绍各个计算 SKU 选项，并提供技术详细信息来帮助你做出最佳选择。

## <a name="select-a-cluster-type"></a>选择群集类型

Azure 数据资源管理器提供两种类型的群集：

* **生产** ：生产群集包含用于引擎和数据管理群集的两个节点，根据 Azure 数据资源管理器的 [SLA](https://www.azure.cn/support/sla/data-explorer/) 运行。

* **开发/测试（无 SLA）** ：开发/测试群集为引擎和数据管理群集提供单个节点。 此群集类型是成本最低的配置，因为它的实例计数较小，且不收取引擎标记费用。 此群集配置不附带 SLA，因为它不提供冗余。

## <a name="compute-sku-types"></a>计算 SKU 类型

对于不同类型的工作负载，Azure 数据资源管理器群集支持多种 SKU。 每个 SKU 均提供不同的 SSD 和 CPU 比率，以帮助客户正确地调整其部署规模，并为其企业分析工作负载构建成本最佳的解决方案。

### <a name="compute-optimized"></a>计算优化

* 提供较高的核心与缓存比率。
* 适用于中小型数据大小的高查询率。
* 可实现低延迟 I/O 的本地 SSD。

### <a name="heavy-compute"></a>计算密集

* 核心与缓存比率高得多的 AMD SKU。
* 可实现低延迟 I/O 的本地 SSD。

### <a name="storage-optimized"></a>存储优化

* 适用于每引擎节点介于 1 TB 到 4 TB 的更大存储。
* 适用于需要存储大量数据，而要求执行更少计算查询的工作负载。
* 某些 SKU 使用附加到引擎节点的高级存储（托管磁盘）而不是本地 SSD 来实现热数据存储。

### <a name="isolated-compute"></a>独立计算

用于运行需要服务器实例级隔离的工作负载的理想 SKU。

## <a name="select-and-optimize-your-compute-sku"></a>选择并优化计算 SKU 

### <a name="select-your-compute-sku-during-cluster-creation"></a>在群集创建过程中选择计算 SKU

创建 Azure 数据资源管理器群集时，请根据计划的工作负荷选择最佳的 VM SKU。

以下属性也可帮助你作出 SKU 选择：
 
| 属性 | 详细信息 |
|---|---
|**可用性**| 并非所有 SKU 都可在所有区域中使用 |
|**每核心每 GB 缓存成本**| 成本高，计算和密集计算均经过优化。 成本低，具有优化了存储的 SKU |
|**预留实例 (RI) 定价**| RI 折扣因区域和 SKU 而异 |  

> [!NOTE]
> 对于 Azure 数据资源管理器群集，与存储和网络相比，计算成本是群集成本中最重要的部分。

### <a name="optimize-your-cluster-compute-sku"></a>优化群集计算 SKU

若要优化群集计算 SKU，请[配置垂直缩放](manage-cluster-vertical-scaling.md#configure-vertical-scaling)。 

有各种计算 SKU 选项可供选择，因此可根据方案的性能和热缓存要求来优化成本。 
* 如果需要为高查询量实现最佳性能，理想的 SKU 是计算优化版本。 
* 如果需要以较低的查询负载查询大量数据，则存储优化的 SKU 可帮助降低成本，同时仍可提供出色的性能。

由于小型 SKU 的每群集实例数受限制，因此最好是使用 RAM 较大的大型 VM。 某些对 RAM 资源需求更高的查询类型（例如使用 `joins` 的查询）需要更多的 RAM。 因此，在考虑缩放选项时，建议纵向扩展到更大的 SKU，而不要通过添加更多的实例进行横向扩展。

## <a name="compute-sku-options"></a>计算 SKU 选项

下表描述了 Azure 数据资源管理器群集 VM 的技术规格：

<!-- mc only, no need to change -->
|**名称**| **类别** | **SSD 大小** | **核心数** | **RAM** | **高级存储磁盘 (1&nbsp;TB)**| **每个群集的最小实例计数** | **每个群集的最大实例计数**
|---|---|---|---|---|---|---|---
|Dev(No SLA) Standard_D11_v2| 计算优化 | 75&nbsp;GB    | 1 | 14&nbsp;GB | 0 | 1 | 1
|Standard_D11_v2| 计算优化 | 75&nbsp;GB    | 2 | 14&nbsp;GB | 0 | 2 | 8 
|Standard_D12_v2| 计算优化 | 150&nbsp;GB   | 4 | 28&nbsp;GB | 0 | 2 | 16
|Standard_D13_v2| 计算优化 | 307&nbsp;GB   | 8 | 56&nbsp;GB | 0 | 2 | 1,000
|Standard_D14_v2| 计算优化 | 614&nbsp;GB   | 16| 112&nbsp;GB | 0 | 2 | 1,000
|Standard_DS13_v2 + 1&nbsp;TB&nbsp;PS| 存储优化 | 1&nbsp;TB | 8 | 56&nbsp;GB | 1 | 2 | 1,000
|Standard_DS13_v2 + 2&nbsp;TB&nbsp;PS| 存储优化 | 2&nbsp;TB | 8 | 56&nbsp;GB | 2 | 2 | 1,000
|Standard_DS14_v2 + 3&nbsp;TB&nbsp;PS| 存储优化 | 3&nbsp;TB | 16 | 112&nbsp;GB | 2 | 2 | 1,000
|Standard_DS14_v2 + 4&nbsp;TB&nbsp;PS| 存储优化 | 4&nbsp;TB | 16 | 112&nbsp;GB | 4 | 2 | 1,000

* 可使用 Azure 数据资源管理器 [ListSkus API](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.kusto.clustersoperationsextensions.listskus?view=azure-dotnet) 查看各区域已更新的计算 SKU 的列表。 
* 详细了解[各种 SKU](/virtual-machines/windows/sizes)。 

## <a name="next-steps"></a>后续步骤

* 随时可以根据不同的需求，通过更改 VM SKU 来[纵向扩展或缩减](manage-cluster-vertical-scaling.md)引擎群集。 

* 可以根据不断变化的需求，[横向缩减或扩展](manage-cluster-horizontal-scaling.md)引擎群集的大小以改变容量。

