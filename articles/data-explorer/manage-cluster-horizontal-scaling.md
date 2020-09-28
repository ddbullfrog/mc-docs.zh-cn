---
title: 在 Azure 数据资源管理器中管理水平缩放（横向扩展）群集以满足需求
description: 本文介绍如何根据不断变化的需求采用相关步骤对 Azure 数据资源管理器群集进行横向缩放。
author: orspod
ms.author: v-tawe
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: how-to
origin.date: 12/09/2019
ms.date: 09/24/2020
ms.openlocfilehash: 556f2b511429dd5f510a266bf6b3bbe9be078c13
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146688"
---
# <a name="manage-cluster-horizontal-scaling-scale-out-in-azure-data-explorer-to-accommodate-changing-demand"></a>在 Azure 数据资源管理器中管理水平缩放（横向扩展）群集以适应不断变化的需求

调整好群集的大小对于 Azure 数据资源管理器的性能至关重要。 静态群集大小可能导致利用不充分或利用过度，二者都不理想。 由于无法绝对准确地预测群集的需求，更好的方法是对群集进行*缩放*，根据不断变化的需求添加或删除容量和 CPU 资源。 

有两个工作流可用于缩放 Azure 数据资源管理器群集： 
* 水平缩放，也称为横向缩放。
* [垂直缩放](manage-cluster-vertical-scaling.md)，也称为纵向缩放。
本文介绍水平缩放工作流。

## <a name="configure-horizontal-scaling"></a>配置水平缩放

借助水平缩放，可以根据预定义的规则和计划自动缩放实例计数。 若要指定群集的自动缩放设置，请执行以下操作：

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集资源。 在“设置”下选择“横向扩展”。  

2. 在“横向扩展”窗口中，选择想要的自动缩放方法：**手动缩放**或**自定义自动缩放**。

### <a name="manual-scale"></a>手动缩放

手动缩放是群集创建过程中的默认设置。 群集有一个不会自动更改的静态容量。 可以使用“实例计数”栏选择静态容量。 群集的缩放保持在该设置，除非你再次进行更改。

   ![手动缩放方法](media/manage-cluster-horizontal-scaling/manual-scale-method.png)

<!-- Optimized autoscale (preview) not available in mc portal -->
<!-- ### Optimized autoscale -->

### <a name="custom-autoscale"></a>自定义自动缩放

使用自定义自动缩放，可以根据指定的指标动态地缩放群集。 下图显示配置自定义自动缩放的流程和步骤。 图后面提供了更多详细信息。

1. 在“自动缩放设置名称”框中输入一个名称，例如“横向扩展: 缓存使用率”。 

   ![缩放规则](media/manage-cluster-horizontal-scaling/custom-autoscale-method.png)

2. 至于“缩放模式”，请选择“基于指标缩放”。 此模式提供动态缩放。 也可选择“缩放为具体实例数”。

3. 选择“+ 添加规则”。

4. 在右侧的“缩放规则”部分，输入每项设置的值。

    **条件**

    | 设置 | 说明和值 |
    | --- | --- |
    | **时间聚合** | 选择聚合条件，例如“平均”。 |
    | **指标名称** | 选择要求缩放操作依赖的指标，例如“缓存使用率”。 |
    | 时间粒度统计信息 | 在 **Average**、**Minimum**、**Maximum** 和 **Sum** 之间选择。 |
    | **运算符** | 选择适当的选项，例如“大于或等于”。 |
    | **阈值** | 选择适当的值。 例如，对于缓存使用率，一开始可以选择 80%。 |
    | **持续时间（分钟）** | 选择适当的时间值，以便系统在计算指标时进行回溯。 一开始可将默认值设置为 10 分钟。 |
    |  |  |

    **操作**

    | 设置 | 说明和值 |
    | --- | --- |
    | **操作** | 选择进行横向缩减或横向扩展的适当选项。 |
    | **实例计数** | 选择在符合指标条件的情况下，需要添加或删除的节点或实例的数目。 |
    | 冷却（分钟） | 选择需要在两项缩放操作之间等待的适当时间间隔。 一开始可将默认值设置为 5分钟。 |
    |  |  |

5. 选择“添加”  。

6. 在左侧的“实例限制”部分，输入每项设置的值。

    | 设置 | 说明和值 |
    | --- | --- |
    | **最低** | 群集在缩放时不管使用率如何都不得低于的实例数。 |
    | **最高** | 群集在缩放时不管使用率如何都不得高于的实例数。 |
    | **默认** | 默认实例数。 在出现资源指标读取问题时使用此设置。 |
    |  |  |

7. 选择“保存” 。

现在已为 Azure 数据资源管理器群集配置了水平缩放。 添加另一个垂直缩放规则。 如果在解决群集缩放问题时需要帮助，请在 Azure 门户中[提交支持请求](https://portal.azure.cn/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。

## <a name="next-steps"></a>后续步骤

* [通过指标监视 Azure 数据资源管理器的性能、运行状况和使用情况](using-metrics.md)
* [管理群集垂直缩放](manage-cluster-vertical-scaling.md)，以便适当地调整群集的大小。
