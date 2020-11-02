---
title: 适用于虚拟机的 Azure 专用主机概述
description: 详细了解如何使用 Azure 专用主机部署虚拟机。
author: rockboyfor
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure
origin.date: 07/28/2020
ms.date: 08/10/2020
ms.testscope: no
ms.testdate: 07/27/2020
ms.author: v-yeche
ms.reviewer: zivr
ms.openlocfilehash: 36ccf1ddb5dd18fd1bd8ddf66643f31b4a105bb3
ms.sourcegitcommit: 1f933e4790b799ceedc685a0cea80b1f1c595f3d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629818"
---
<!--Verified successfully-->
# <a name="azure-dedicated-hosts"></a>Azure 专用主机

<!--Not Avaialble on availability zone-->

Azure 专用主机是一种提供物理服务器（能够托管一个或多个虚拟机）的服务，专用于一个 Azure 订阅。 专用主机是数据中心中使用的相同物理服务器，作为资源提供。 你可以在区域和容错域中预配专用主机。 然后，可以按照最能满足你的需要的配置将 VM 直接放入预配的主机中。

<!--Include file updated-->

[!INCLUDE [virtual-machines-common-dedicated-hosts](../../../includes/virtual-machines-common-dedicated-hosts.md)]

## <a name="next-steps"></a>后续步骤

- 可使用 [Azure PowerShell](dedicated-hosts-powershell.md)、[门户](dedicated-hosts-portal.md)和 [Azure CLI](../linux/dedicated-hosts-cli.md) 部署专用主机。

- [此处](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md)有一个示例模板，该模板使用区域和容错域来最大限度地提高在某个地区的复原能力。

<!--Not Available on [Reserved Instance of Azure Dedicated Hosts](../prepay-dedicated-hosts-reserved-instances.md)-->

<!-- Update_Description: update meta properties, wording update, update link -->