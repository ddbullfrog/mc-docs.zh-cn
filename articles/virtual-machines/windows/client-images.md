---
title: 在 Azure 中使用 Windows 客户端映像
description: 如何使用 Visual Studio 订阅权益，在 Azure 中针对开发/测试方案部署 Windows 7、Windows 8 或 Windows 10
ms.subservice: imaging
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure-services
origin.date: 12/15/2017
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: e8814be84c7ca3348cb1cc47af1cebde79e9a568
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104708"
---
<!--Verified successfully-->
# <a name="use-windows-client-in-azure-for-devtest-scenarios"></a>在 Azure 中使用 Windows 客户端实现开发/测试方案
如果有相应的 Visual Studio（以前为 MSDN）订阅，可以在 Azure 中使用 Windows 7 或 Windows 10 企业版 (x64) 实施开发/测试方案。 本文概述在 Azure 中运行 Windows 7 或 Windows 10 企业版以及使用以下 Azure 库映像所要满足的条件。

<!--Not Available on Windows 8-->

:::image type="content" source="./media/client-images/windows-client-msdn-images.png" alt-text="Azure 门户中的映像详细信息"::: 

> [!NOTE]
> 有关 Azure 库中的 Windows 10 专业版和 Windows 10 专业版 N 映像，请参阅[如何使用多租户托管权限在 Azure 上部署 Windows 10](windows-desktop-multitenant-hosting-deployment.md)
>:::image type="content" source="./media/client-images/windows-client-pro-images.png" alt-text="Azure 门户中的映像详细信息"::: 
>

## <a name="subscription-eligibility"></a>订阅条件
有效的 Visual Studio 订户（已获取 Visual Studio 订阅许可证的用户）可以使用 Windows 客户端进行开发和测试。 可以在自己的硬件上，以及在任何类型的 Azure 订阅中运行的 Azure 虚拟机上使用 Windows 客户端。 不能在 Azure 上针对一般生产目的部署或使用 Windows 客户端，不是有效 Visual Studio 订户的用户也不能使用 Windows 客户端。

为方便起见，在 Azure 库的 [符合条件的开发/测试产品](#eligible-offers)中提供了某些 Windows 10 映像。 属于任一产品类型的 Visual Studio 订户也可以[适当地准备和创建](prepare-for-upload-vhd-image.md) 64 位 Windows 7、Windows 8 或 Windows 10 映像，并[上载到 Azure](upload-generalized-managed.md)。 仅限有效的 Visual Studio 订户用于开发/测试目的。

## <a name="eligible-offers"></a>符合条件的产品
下表详细描述了可通过 Azure 库部署 Windows 10 的产品 ID。 只有在以下产品中才能看到 Windows 10 映像。 需要以其他产品类型运行 Windows 客户端的 Visual Studio 订阅者需要[适当地准备和创建](prepare-for-upload-vhd-image.md) 64 位 Windows 7 或 Windows 10 映像，[并上传到 Azure](upload-generalized-managed.md)。

<!--Not Available on Windows 8-->

<!--MOONCAKE CUSTOMIZE ON 09/04/2020-->

| 产品名称 | 产品编号 | 可用客户端映像 |
|:--- |:---:|:---:|
| [Azure 标准预付费](https://www.azure.cn/offers/ms-mc-arz-33p/) |MS-MC-AZR-33P|Windows 10 |
| [Visual Studio Professional（订阅者权益）](https://www.azure.cn/offers/ms-mc-arz-msdn/) |MS-MC-AZR-59P |Windows 10 |
| [Visual Studio Test Professional（订阅者权益）](https://www.azure.cn/offers/ms-mc-arz-msdn/) |MS-MC-AZR-60P |Windows 10 |
| [MSDN 平台（订阅者权益）](https://www.azure.cn/offers/ms-mc-arz-msdn/) |MS-MC-AZR-62P |Windows 10 |
| [Visual Studio Enterprise（订阅者权益）](https://www.azure.cn/offers/ms-mc-arz-msdn/) |MS-MC-AZR-63P |Windows 10 |

<!--MOONCAKE CUSTOMIZE ON 09/04/2020-->

## <a name="check-your-azure-subscription"></a>检查 Azure 订阅
如果不知道自己的产品 ID，可以使用以下方式通过 Azure 门户获取：  

- 在“订阅”窗口中：

    :::image type="content" source="./media/client-images/offer-id-azure-portal.png" alt-text="Azure 门户中的映像详细信息"::: 

    <!--Not Available on - Or, click **Billing** and then click your subscription ID. The offer ID appears in the *Billing* window.-->

也可以从 Azure 帐户门户的[“订阅”选项卡](https://account.windowsazure.cn/Subscriptions)查看套餐 ID：

:::image type="content" source="./media/client-images/offer-id-azure-account-portal.png" alt-text="Azure 门户中的映像详细信息"::: 

## <a name="next-steps"></a>后续步骤
现在，可以使用 [PowerShell](quick-create-powershell.md)、[Resource Manager 模板](ps-template.md)或 [Visual Studio](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) 部署 VM。

<!-- Update_Description: update meta properties, wording update, update link -->