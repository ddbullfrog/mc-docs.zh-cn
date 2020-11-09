---
title: 管理资源和配额
titleSuffix: Azure Machine Learning
description: 了解 Azure 机器学习资源的配额以及如何请求更多配额。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
author: nishankgu
ms.author: nigup
ms.date: 10/13/2020
ms.topic: conceptual
ms.custom: troubleshooting,contperfq4, contperfq2
ms.openlocfilehash: e1569bc28855bd414e1d3a7561dc9ba009fa8609
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103538"
---
# <a name="manage--increase-quotas-for-resources-with-azure-machine-learning"></a>管理和增加 Azure 机器学习资源的配额

Azure 使用限制和配额来防止由于欺诈导致的预算超支，并遵循 Azure 容量约束。 对于生产工作负荷，在缩放时请考虑这些限制。 本文介绍：

> [!div class="checklist"]
> + 与 [Azure 机器学习](overview-what-is-azure-ml.md)相关的 Azure 资源的默认限制。
> + 查看你的配额和限制。
> + 创建工作区级别的配额。
> + 请求增加配额。
> + 专用终结点和 DNS 配额。

## <a name="special-considerations"></a>特殊注意事项

+ 配额是一种信用限制，不附带容量保证。 如果有大规模容量需求，[请与 Azure 支持部门联系来增加你的配额](#request-quota-increases)。

+ 配额在订阅中的所有服务（包括 Azure 机器学习）之间共享。 评估容量时会计算所有服务中的使用量。
    + Azure 机器学习计算是一个例外，它的配额独立于核心计算配额。 

+ 默认限制根据产品/服务类别类型（例如免费试用、即用即付）和虚拟机 (VM) 系列（例如 Dv2、F、G 等）而有所不同。

## <a name="default-resource-quotas"></a>默认资源配额

在本部分中，你将了解以下资源的默认和最大配额限制：

+ 虚拟机
+ Azure 机器学习计算
+ Azure 机器学习管道
+ 容器实例
+ 存储

> [!IMPORTANT]
> 限制随时会变化。 始终可以在所有 Azure 的服务级别配额[文档](/azure-resource-manager/management/azure-subscription-service-limits/)中找到最新的限制。

### <a name="virtual-machines"></a>虚拟机
每个 Azure 订阅都对所有服务中的虚拟机数量进行了限制。 虚拟机核心数既有区域总数限制，又有按大小系列（Dv2、F 等）的区域限制。 这两种限制单独实施。

例如，假设某个订阅的美国东部 VM 核心总数限制为 30，A 系列核心数限制为 30，D 系列核心数限制为 30。 该订阅可以部署 30 个 A1 VM、30 个 D1 VM，或者两者的组合，但其总数不能超过 30 个核心。

对虚拟机的限制不能高于下表中显示的值。

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../../includes/azure-subscription-limits-azure-resource-manager.md)]

### <a name="azure-machine-learning-compute"></a>Azure 机器学习计算
在 [Azure 机器学习计算](concept-compute-target.md#azure-machine-learning-compute-managed)中，订阅中每个区域所允许的核心数和唯一计算资源数都有默认配额限制。 此配额与上一部分中的 VM 核心配额不同。

[请求增加配额](#request-quota-increases)，将此部分中的限制提高到表中所示的 **最大限制** 。

可用资源：
+ **每个区域的专用核心数** 的默认限制为 24 - 300 个，具体取决于订阅套餐的类型。  可以为每个 VM 系列提高每个订阅的专用核心数。 专业化 VM 系列（例如 NCv2、NCv3 或 ND 系列）最初的默认限制为零个核心。

+ **每个区域的低优先级核心数** 的默认限制为 100 - 3000 个，具体取决于订阅套餐的类型。 每个订阅的低优先级核心数可以提高，对不同的 VM 系列采用单个值。

+ **每个区域的群集数** 的默认限制为 200。 该数字在训练群集与计算实例（在配额消耗中被视为单节点群集）之间共享。

下表显示了不能超过的其他限制。

| **资源** | **最大限制** |
| --- | --- |
| 每个资源组的工作区数 | 800 |
| 单个 Azure 机器学习计算 (AmlCompute) 资源中的节点数 | 100 个节点 |
| 每个节点的 GPU MPI 进程数 | 1-4 |
| 每个节点的 GPU 辅助角色数 | 1-4 |
| 作业生存期 | 21 天<sup>1</sup> |
| 低优先级节点上的作业生存期 | 7 天<sup>2</sup> |
| 每个节点的参数服务器数 | 1 |

<sup>1</sup> 最大生存期是指从运行开始到运行完成之间的持续时间。 已完成的运行无限期保留。 最长生存期内未完成的运行的数据不可访问。
<sup>2</sup> 每当存在容量约束时，低优先级节点上的作业可能会预先清空。 我们建议在作业中实施检查点。

### <a name="azure-machine-learning-pipelines"></a>Azure 机器学习管道
[Azure 机器学习管道](concept-ml-pipelines.md)具有以下限制。

| **资源** | **限制** |
| --- | --- |
| 管道中的步骤 | 30,000 |
| 每个资源组的工作区数 | 800 |

### <a name="container-instances"></a>容器实例

有关详细信息，请参阅[容器实例限制](/azure-resource-manager/management/azure-subscription-service-limits#container-instances-limits)。

### <a name="storage"></a>存储
Azure 存储帐户的限制是每个订阅在每个区域中的存储帐户数不能超过 250 个。 这包括标准和高级存储帐户。

若要提高此限制，请通过 [Azure 支持](https://ms.portal.azure.cn/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/)提交请求。 Azure 存储团队会评审你的案例，最多可以为每个区域批准 250 个存储帐户。


## <a name="workspace-level-quota"></a>工作区级别配额

使用工作区级配额来管理同一订阅中多个[工作区](concept-workspace.md)之间的 Azure 机器学习计算目标分配。

默认情况下，所有工作区的配额与任何 VM 系列的订阅级配额相同。 但是，你可以在订阅中的工作区上设置各个 VM 系列的最大配额。 这使你可以共享容量，避免资源争用问题：

1. 导航到你的订阅中的任何工作区。
1. 在左侧窗格中，选择“使用量 + 配额”。
1. 选择“配置配额”选项卡以查看配额。
1. 展开某个 VM 系列。
1. 在任何工作区（在该 VM 系列下列出）上设置配额限制。

不能设置负值或大于订阅级配额的值。

[![Azure 机器学习工作区级别的配额](./media/how-to-manage-quotas/azure-machine-learning-workspace-quota.png)](./media/how-to-manage-quotas/azure-machine-learning-workspace-quota.png)

> [!NOTE]
> 需要拥有订阅级别的权限才能在工作区级别设置配额。

## <a name="view-your-usage-and-quotas"></a>查看使用情况和配额

若要查看各种 Azure 资源（例如虚拟机、存储、网络）的配额，请使用 Azure 门户：

1. 在左窗格上，选择“所有服务”，然后在“一般”类别下选择“订阅” 。

2. 从订阅列表中选择要查找其配额的订阅。

3. 选择“使用情况 + 配额”以查看当前的配额限制和使用情况。 使用筛选器选择提供者和位置。 

订阅中的 Azure 机器学习计算配额与其他 Azure 配额分开管理。 

1. 在 Azure 门户中导航到你的 **Azure 机器学习** 工作区。

2. 在左侧窗格中的“支持 + 故障排除”部分下，选择“使用情况 + 配额”以查看当前的配额限制和使用情况 。

3. 选择订阅以查看配额限制。 请记住筛选到所需的区域。

4. 可以在订阅级视图与工作区级视图之间切换：

## <a name="request-quota-increases"></a>请求增加配额

若要提高限制或配额，使其超出默认限制，可以免费[提交联机客户支持请求](https://ms.portal.azure.cn/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/)。

无法将限制提高到高于上述表中所示的最大限制值。 如果没有最大限制，则无法调整资源的限制。

请求增加配额时，请选择要为其请求提升配额的服务。 例如，Azure 机器学习、容器实例、存储等。对于 Azure 机器学习计算，可以在按照上述步骤查看配额时选择“请求配额”按钮。

> [!NOTE]
> [试用订阅](https://www.azure.cn/pricing/1rmb-trial/)没有资格提高限制或配额。 如果使用的是[试用订阅](https://www.azure.cn/pricing/1rmb-trial/)，可将其升级到[即用即付](https://wd.azure.cn/zh-cn/pricing/pia-waiting-list/?form-type=identityauth)订阅。 
## <a name="private-endpoint-and-private-dns-quota-increases"></a>专用终结点和专用 DNS 配额增加

可在订阅中创建的专用终结点和专用 DNS 区域的数目存在限制。

虽然 Azure 机器学习在你的（客户）订阅中创建资源，但在某些情况下，会在 Microsoft 拥有的订阅中创建资源。

 在以下方案中，你可能需要在 Microsoft 拥有的订阅中请求配额宽限：

* __使用客户管理的密钥 (CMK) 启用专用链接的工作区__
* __虚拟网络后的工作区的 Azure 容器注册表__
* __将启用了专用链接的 Azure Kubernetes 服务群集附加到你的工作区__ 。

若要针对这些方案请求宽限，请使用以下步骤：

1. [创建 Azure 支持请求](/azure-portal/supportability/how-to-create-azure-support-request#create-a-support-request)并从“基本信息”部分中选择以下选项：

    | 字段 | 选择 |
    | ----- | ----- |
    | 问题类型 | 技术 |
    | 服务 | 我的服务。 在下拉列表中选择“机器学习”。 |
    | 问题类型 | 工作区设置、SDK 和 CLI |
    | 问题子类型 | 预配或管理工作区时出现问题 |

2. 在“详细信息”部分中，使用“说明”字段提供要使用的 Azure 区域以及计划使用的方案。 如果需要为多个订阅请求增加配额，还请在此字段中列出订阅 ID。

3. 选择“创建”以创建请求。


