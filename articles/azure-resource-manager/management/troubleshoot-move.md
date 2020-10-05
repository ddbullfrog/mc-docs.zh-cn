---
title: 排查迁移错误
description: 排查将资源移到新资源组或订阅时出现的问题。
ms.topic: conceptual
origin.date: 08/27/2019
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: ffbeec842f2bc8b3d4efcaa0b5d34b02f5306e8b
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146769"
---
<!--Verified successfully-->
# <a name="troubleshoot-moving-azure-resources-to-new-resource-group-or-subscription"></a>排查将 Azure 资源移到新的资源组或订阅时遇到的问题

本文提供了一些建议来帮助解决在移动资源时遇到的问题。

## <a name="upgrade-a-subscription"></a>升级订阅

如果确实想要升级 Azure 订阅（例如从免费切换到标准的预先支付套餐），则需要转换订阅。

<!--MOONCAKE: CORRECT ON FOLLOWING UPGRADE SUBSCRIPTION-->

* 若要升级试用版，请参阅[将试用版或 Azure Imagine Azure 订阅升级为标准预付费套餐](https://docs.azure.cn/billing/billing-upgrade-1rmb-trial/)。
* 若要更改标准预付费套餐帐户，请参阅[将 Azure 标准预付费套餐订阅更改为其他套餐](https://docs.azure.cn/billing/billing-upgrade-to-ea/)。

<!--MOONCAKE: CORRECT ON /billing/billing-upgrade-1rmb-trial AND /billing/billing-upgrade-to-ea-->

如果无法转换订阅，请[创建 Azure 支持请求](https://support.azure.cn/support/support-azure/)。 选择“订阅管理”  作为问题类型。

## <a name="service-limitations"></a>服务限制

有些服务在移动资源时需要注意额外的事项。 如果要移动以下服务，请务必查看指南和限制。

* [应用服务](./move-limitations/app-service-move-limitations.md)
    
    <!--Not Available on * [Azure DevOps Services](https://docs.microsoft.com/azure/devops/organizations/billing/change-azure-subscription?toc=/azure-resource-manager/toc.json)-->

* [经典部署模型](./move-limitations/classic-model-move-limitations.md)
* [联网](./move-limitations/networking-move-limitations.md)

    <!--Not Available on * [Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure-resource-manager/toc.json)-->

* [虚拟机](./move-limitations/virtual-machines-move-limitations.md)

## <a name="large-requests"></a>大型请求

在可能的情况下，将大型移动分为单独的移动操作。 在一次操作中有 800 多项资源时，资源管理器会立即返回错误。 但是，移动 800 项以下的资源也可能因超时而失败。

## <a name="resource-not-in-succeeded-state"></a>资源未处于成功状态

如果收到一条错误消息，指出由于资源未处于成功状态而无法移动资源，则它可能实际上是阻止移动的依赖资源。 通常，错误代码为 **MoveCannotProceedWithResourcesNotInSucceededState**。

如果源或目标资源组包含虚拟网络，则会在移动过程中检查虚拟网络的所有依赖资源的状态。 该检查包括直接和间接依赖于虚拟网络的这些资源。 如果这些资源中有任何资源处于故障状态，则会阻止移动。 例如，如果某个使用虚拟网络的虚拟机故障，则会阻止移动。 即使该虚拟机不是要移动的资源之一，也不在要移动的资源组之一中，系统也会阻止移动。

收到此错误时，你有两个选择。 将资源移到没有虚拟网络的资源组，或[联系支持人员](https://support.azure.cn/support/support-azure/)。

## <a name="next-steps"></a>后续步骤

有关用于移动资源的命令，请参阅[将资源移到新资源组或订阅](move-resource-group-and-subscription.md)。

<!-- Update_Description: update meta properties, wording update, update link -->