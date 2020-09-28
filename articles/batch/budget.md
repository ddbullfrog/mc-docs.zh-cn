---
title: 成本分析和预算
description: 了解如何进行成本分析，并为用于运行 Batch 工作负荷的基础计算资源和软件许可证设置预算。
ms.topic: how-to
ms.service: batch
origin.date: 07/19/2019
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: no
ms.testdate: 04/26/2020
ms.author: v-yeche
ms.openlocfilehash: 23ccb496dad4c737b7ea37512e724aa1e18012ea
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146629"
---
# <a name="cost-analysis-and-budgets-for-azure-batch"></a>Azure Batch 成本分析和预算

Azure Batch 本身是免费的，只有用于运行 Batch 工作负荷的基础计算资源和软件许可证是收费的。 概括来讲，成本来自池中的虚拟机 (VM)、从 VM 进行的数据传输或在云中存储的任何输入或输出数据。 接下来将介绍 Batch 的一些关键组件，以了解成本从何而来、如何为池或帐户设置预算，以及让 Batch 工作负荷更经济高效的一些技术。

## <a name="batch-resources"></a>Batch 资源

虚拟机是用于 Batch 处理的最重要资源。 Batch 的 VM 使用成本是根据类型、数量和使用持续时间计算的。 VM 计费选项包括[标准预付费套餐](https://wd.azure.cn/pricing/pia-waiting-list/)或 [MSDN 订阅 Azure 权益套餐](https://www.azure.cn/offers/ms-mc-arz-msdn/index.html)。 这两种支付选项的优势因计算工作负荷而异，而且这两种付款模型对帐单的影响也不同。

<!--CORRECT ON [MSDN Subscriptions Azure Benefit Offer](https://www.azure.cn/offers/ms-mc-arz-msdn/index.html)-->
<!--Not Available on [reservation](../cost-management-billing/reservations/save-compute-costs-reservations.md)-->

使用[应用包](batch-application-packages.md)将应用部署到 Batch 节点 (VM) 时，还需要为应用包使用的 Azure 存储资源付费。 你还要为任何输入或输出文件（如资源文件和其他日志数据）的存储支付费用。 通常，与 Batch 关联的存储数据的成本要比计算资源的成本低得多。 池中使用 VirtualMachineConfiguration 创建的每个 VM 都有一个使用 Azure 托管磁盘的关联 OS 磁盘。 Azure 托管磁盘有额外的成本，其他磁盘性能层也有不同的成本。

Batch 池使用网络资源。 具体而言，对于 VirtualMachineConfiguration 池，使用了需要静态 IP 地址的标准负载均衡器。 Batch 使用的负载均衡器对“用户订阅”帐户可见，但对“Batch 服务”帐户不可见。 传入和传出 Batch 池 VM 的所有数据都会让标准负载均衡器产生费用；选择从池节点检索数据的 Batch API（如“获取任务/节点文件”）、任务应用包、资源/输出文件和容器映像会产生费用。

### <a name="additional-services"></a>其他服务

在计算 Batch 帐户的成本时，可能需要考虑不包括 VM 和存储的服务这一因素。

通常用于 Batch 的其他服务可能包括：

- Application Insights
- 数据工厂
- Azure Monitor
- 虚拟网络
- 带有图形应用的 VM

根据要将哪些服务用于 Batch 解决方案，可能会产生额外的费用。 若要确定每个附加服务的成本，请参阅[定价计算器](https://www.azure.cn/pricing/calculator/)。

<!--Not Available on ## Cost analysis and budget for a pool-->

## <a name="minimize-cost"></a>最大限度地降低成本

长时间使用多个 VM 和 Azure 服务的成本可能会很高。 幸运的是，有一些服务可以帮助你减少开支，也有一些策略可以帮助你最大限度地提升工作负荷的效率。

<!--Not Available on ### Low-priority virtual machines-->

### <a name="virtual-machine-os-disk-type"></a>虚拟机 OS 磁盘类型

有多种 [VM OS 磁盘类型](../virtual-machines/disks-types.md)。 大多数 VM 系列大小都支持高级和标准存储。 为池选择“s”VM 大小时，Batch 会配置高级 SSD OS 磁盘。 选择“非 s”VM 大小时，会使用更经济的标准 HDD 磁盘类型。 例如，将高级 SSD OS 磁盘用于 `Standard_D2s_v3`，将标准 HDD OS 磁盘用于 `Standard_D2_v3`。

高级 SSD OS 磁盘更昂贵，但具有更高的性能，并且包含高级磁盘的 VM 的启动速度比包含标准 HDD OS 磁盘的 VM 稍微快一些。 在 Batch 中，OS 磁盘通常不会频繁使用，因为应用程序和任务文件位于 VM 的临时 SSD 磁盘中。 因此，在许多情况下，如果指定了“s”VM 大小，则无需为预配的高级 SSD 支付更高的费用。

<!--Not Available on ### Reserved virtual machine instances-->
<!--Not Available on [Azure Reservations](../cost-management-billing/reservations/save-compute-costs-reservations.md)-->

### <a name="automatic-scaling"></a>自动缩放

[自动缩放](batch-automatic-scaling.md)根据当前作业的需求来动态缩放 Batch 池中的 VM 数。 通过根据作业生存期来缩放池，自动缩放可确保 VM 仅在有作业需要执行时才进行纵向扩展和使用。 当作业完成或没有作业时，VM 就会自动纵向缩减来节省计算资源。 借助缩放，可以只使用所需的资源，从而降低 Batch 解决方案的总体成本。

若要详细了解自动缩放，请参阅[自动缩放 Azure Batch 池中的计算节点](batch-automatic-scaling.md)。

## <a name="next-steps"></a>后续步骤

- 详细了解可用于生成和监视 Batch 解决方案的 [Batch API 和工具](batch-apis-tools.md)。  

<!--Not Available on - Learn about [low-priority VMs with Batch](batch-low-pri-vms.md)-->

<!-- Update_Description: update meta properties, wording update, update link -->