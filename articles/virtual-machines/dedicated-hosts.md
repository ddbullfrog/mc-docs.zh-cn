---
title: 适用于虚拟机的 Azure 专用主机概述
description: 详细了解如何使用 Azure 专用主机部署虚拟机。
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure
origin.date: 07/28/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: 07/27/2020
ms.author: v-yeche
ms.reviewer: zivr
ms.openlocfilehash: c7f4991989e96432a2ac3fd01d5ada8aadbebcfb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106134"
---
<!--Verified successfully from release articles-->
# <a name="azure-dedicated-hosts"></a>Azure 专用主机

<!--Not Avaialble on availability zone-->

Azure 专用主机是一种提供物理服务器（能够托管一个或多个虚拟机）的服务，专用于一个 Azure 订阅。 专用主机是数据中心中使用的相同物理服务器，作为资源提供。 你可以在区域和容错域中预配专用主机。 然后，可以按照最能满足你的需要的配置将 VM 直接放入预配的主机中。


## <a name="benefits"></a>优点 

预留整个主机具有以下优势：

- 物理服务器级别的硬件隔离。 不会在你的主机上放置任何其他 VM。 专用主机部署在相同的数据中心，与其他非隔离主机共享相同的网络和底层存储基础结构。
- 控制 Azure 平台发起的维护事件。 尽管大多数维护事件对虚拟机的影响极小甚至没有任何影响，但对于某些敏感工作负荷而言，每暂停一秒都可能会造成影响。 在使用专用主机的情况下，可以选择维护时段，以降低对服务的影响。
- 借助 Azure 混合权益，你可以在 Azure 中使用自己的 Windows 和 SQL 许可证。 使用混合权益可以提供更多的优势。 有关详细信息，请参阅 [Azure 混合权益](https://www.azure.cn/pricing/hybrid-benefit/)。

## <a name="groups-hosts-and-vms"></a>组、主机和 VM  

:::image type="content" source="./media/virtual-machines-common-dedicated-hosts/dedicated-hosts2.png" alt-text="专用主机的新资源的视图。":::

主机组  是表示专用主机集合的资源。 你在某个区域中创建主机组，并向其中添加主机。

<!--Not Available on and an availability zone-->

主机  是映射到 Azure 数据中心内的物理服务器的资源。 创建主机时，将分配物理服务器。 主机是在主机组中创建的。 主机提供一个 SKU，用于描述可创建哪些 VM 大小。 每个主机可以托管多个不同大小的 VM，前提是这些 VM 来自相同的大小系列。

## <a name="high-availability-considerations"></a>高可用性注意事项 

要实现高可用性，应部署多个 VM，并将其分散在多个主机（最少 2 个）上。 在使用 Azure 专用主机的情况下，有多个选项可用来预配基础结构，以形成故障隔离边界。

<!--Not Available on ### Use Availability Zones for fault isolation-->

### <a name="use-fault-domains-for-fault-isolation"></a>使用容错域实现故障隔离

可以在特定的容错域中创建主机。 与规模集或可用性集中的 VM 一样，不同容错域中的主机将放置在数据中心内的不同物理机架上。 创建主机组时，需要指定容错域计数。 在主机组中创建主机时，需为每个主机分配容错域。 VM 不需要任何容错域分配。

容错域不同于归置。 对两个主机使用同一个容错域并不意味着它们彼此邻近。

容错域的范围限定为主机组。 不应该对两个主机组之间的反关联做出任何假设。

<!--Not Available on (unless they are in different availability zones)-->

部署到具有不同容错域的主机的 VM 将在多个存储戳上具有其底层托管磁盘服务，以提高故障隔离保护。

<!--Not Available on ### Using Availability Zones and Fault Domains-->

## <a name="manual-vs-automatic-placement"></a>手动放置与自动放置 

> [!IMPORTANT]
> 自动放置目前以公共预览版提供。
> 若要参与使用该预览版，请在 [https://aka.ms/vmss-adh-preview](https://aka.ms/vmss-adh-preview) 上完成关于加入预览的调查。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅[适用于 Azure 预览版的补充使用条款](https://www.azure.cn/support/legal/subscription-agreement/)。

在 Azure 中创建 VM 时，可以选择要使用哪个专用主机。 还可以使用相应的选项将 VM 自动放置在主机组中的现有主机上。 

在创建新主机组时，请确保选择用于自动 VM 放置的设置。 在创建 VM 时，请选择该主机组，并让 Azure 为 VM 选择最佳主机。 

已启用自动放置的主机组并不要求所有的 VM 均自动放置。 即使为该主机组选择了自动放置，你仍然能够显式选择主机。 

### <a name="limitations"></a>限制

在使用自动 VM 放置时，有以下已知的问题和限制：

- 无法在专用主机上应用 Azure 混合权益。
- 无法重新部署 VM。 
- 无法控制专用主机的维护。
- 无法将 Msv2 或 M 系列 VM 用于专用主机 

<!--Not Available on Lsv2, NVasv4, NVsv3,-->

## <a name="virtual-machine-scale-set-support"></a>虚拟机规模集支持

利用虚拟机规模集，可以将一组虚拟机视为单一资源，并可以将其作为一个组来应用可用性、管理、缩放和业务流程策略。 现有的专用主机也可以用于虚拟机规模集。 

> [!IMPORTANT]
> 专用主机上的虚拟机规模集目前以公共预览版提供。
> 若要参与使用该预览版，请在 [https://aka.ms/vmss-adh-preview](https://aka.ms/vmss-adh-preview) 上完成关于加入预览的调查。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅[适用于 Azure 预览版的补充使用条款](https://www.azure.cn/support/legal/subscription-agreement/)。

在创建虚拟机规模集时，可以指定现有主机组，以便在专用主机上创建所有的 VM 实例。

在专用主机组中创建虚拟机规模集时，需要满足以下要求：

- 需要启用自动 VM 放置。
- 主机组的可用性设置应当与规模集相匹配。 
    - 对于区域规模集，应该使用区域主机组。
    
    <!--Not Available on (created without specifying an availability zone)-->
    <!--Not Available on - The host group and the scale set must be using the same availability zone.-->
    
    - 主机组级别的容错域计数应当与规模集的容错域计数相匹配。 通过 Azure 门户，可以为规模集指定“最大分布”，这会将容错域计数设置为 1。
- 应该先创建具有足够容量且具有相同规模集区域设置与容错域设置的专用主机。
- 专用主机支持的 VM 大小应当与用于规模集的 VM 大小相匹配。

专用主机并非支持所有的规模集业务流程和优化设置。 请将以下设置应用于规模集： 
- 禁用过度预配。
- 使用 ScaleSetVM 业务流程模式 
- 请勿将邻近放置组用于归置

## <a name="maintenance-control"></a>维护控制

可以偶尔更新支持虚拟机的基础结构，以提高可靠性、性能和安全性，并启动新功能。 只要有可能，Azure 平台就会尝试最大程度地降低平台维护的影响，但如果客户具有对维护敏感的  工作负荷，那么他们就不能容忍出于维护目的而冻结 VM 或断开 VM 连接，哪怕只有几秒。

维护控制  为客户提供了一个选项，以用于跳过在其专用主机上计划的定期平台更新，然后在 35 天滚动时段内他们所选的时间应用更新。

有关详细信息，请参阅[通过维护控制管理平台更新](./maintenance-control.md)。

## <a name="capacity-considerations"></a>容量注意事项

预配了专用主机之后，Azure 会将其分配给物理服务器。 这会在你需要预配 VM 时保证容量的可用性。 Azure 使用区域中的全部容量为你的主机选取物理服务器。 这也意味着，客户可以预期其专用主机占用空间能够扩展，而不必担心用尽群集中的空间。

<!--Not Available on (or zone)-->

## <a name="quotas"></a>配额

部署专用主机时会使用两种类型的配额。

1. 专用主机 vCPU 配额。 默认配额限制为每个区域 3000 vCPU。
1. VM 大小系列配额。 例如，在中国东部区域，标准预付费套餐  订阅仅为 Dsv3 大小系列提供 10 个 vCPU 的配额。 若要部署 Dsv3 专用主机，则需要请求将配额提高到至少 64 个 vCPU，然后才能部署专用主机。 

若要请求提高配额，请在 [Azure 门户](https://support.azure.cn/support/support-azure/)中创建支持请求。

预配专用主机时将同时使用专用主机 vCPU 和 VM 系列 vCPU 配额，但不会使用区域 vCPU。

:::image type="content" source="./media/virtual-machines-common-dedicated-hosts/quotas.png" alt-text="门户中的用量和配额页的屏幕截图":::

有关详细信息，请参阅[虚拟机 vCPU 配额](./windows/quotas.md)。

试用版和 MSDN 订阅不为 Azure 专用主机提供配额。

## <a name="pricing"></a>定价

无论部署多少个 VM，都会按专用主机对用户收费。 在每月帐单中，你将看到新的可计费资源类型的主机。 专用主机上的 VM 仍会显示在帐单中，但其价格为 0。

主机价格是根据 VM 系列、类型（硬件大小）和区域设置的。 主机价格与主机上支持的最大 VM 大小相关。

软件许可、存储和网络用量与主机和 VM 分开计费。 这些可计费项没有更改。

有关详细信息，请参阅 [Azure 专用主机定价](https://www.azure.cn/pricing/details/virtual-machines/)。

<!--Not Available on [Reserved Instance of Azure Dedicated Hosts](prepay-dedicated-hosts-reserved-instances.md)-->

## <a name="sizes-and-hardware-generations"></a>大小和硬件代系

SKU 是为主机定义的，表示 VM 大小系列和类型。 可以在单个主机中混合使用多个不同大小的 VM，只要这些 VM 属于相同的大小系列即可。 

类型  为硬件代系。 相同 VM 系列的不同硬件类型来自不同的 CPU 供应商，具有不同的 CPU 代系和核心数。 

大小和硬件类型因区域而异。 请参阅主机[定价页](https://www.azure.cn/pricing/details/virtual-machines/)来了解详细信息。

## <a name="host-life-cycle"></a>主机生命周期

Azure 将会监视和管理主机的运行状况状态。 查询主机时，将返回以下状态：

| 运行状况状态   | 说明       |
|----------|----------------|
| 主机可用     | 主机不存在已知问题。   |
| 主机正在接受调查  | 主机出现了一些问题，我们正在调查。 这是一种过渡状态，需要 Azure 尝试识别所发现的问题的范围和根本原因。 主机上运行的虚拟机可能受到了影响。 |
| 主机正在等待解除分配   | Azure 无法将主机还原到正常状态，并要求你将虚拟机重新部署到此主机外部。 如果已启用 `autoReplaceOnFailure`，则虚拟机会将服务修复  到正常的硬件上。 否则，虚拟机可能在即将发生故障的主机上运行。|
| 主机已解除分配  | 所有虚拟机都已从主机中删除。 由于硬件已退出轮换，因此不再向你收取此主机的费用。   |

## <a name="next-steps"></a>后续步骤

- 可使用 [Azure PowerShell](./windows/dedicated-hosts-powershell.md)、[门户](./dedicated-hosts-portal.md)和 [Azure CLI](./linux/dedicated-hosts-cli.md) 部署专用主机。

- [此处](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md)有一个示例模板，该模板使用区域和容错域来最大限度地提高在某个地区的复原能力。

<!--Not Available on [Reserved Instance of Azure Dedicated Hosts](prepay-dedicated-hosts-reserved-instances.md)-->

<!-- Update_Description: update meta properties, wording update, update link -->
