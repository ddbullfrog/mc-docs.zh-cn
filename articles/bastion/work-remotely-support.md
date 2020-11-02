---
title: 使用 Bastion 进行远程工作 - Azure Bastion
description: 本页介绍在 COVID-19（新冠病毒）疫情期间如何利用 Azure Bastion 来实现远程工作。
services: bastion
ms.service: bastion
ms.topic: conceptual
origin.date: 03/25/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 114289e224d8772d39e872a5a86cfae62da19c70
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104966"
---
<!--Verified successfully-->
<!--Character contents only-->
# <a name="working-remotely-using-azure-bastion"></a>使用 Azure Bastion 进行远程工作

Azure Bastion 通过允许具有 Internet 连接的用户访问 Azure 虚拟机，在支持远程工作方案方面起着关键作用。 特别是，它使 IT 管理员能够在全球各地随时随地管理他们在 Azure 上运行的应用程序。

>[!NOTE]
>本文介绍如何利用 Azure Bastion、Azure、Azure 网络和 Azure 合作伙伴生态系统进行远程工作，以及如何缓解因 COVID-19（新冠病毒）危机而面临的网络问题。
>

## <a name="securely-access-virtual-machines"></a>安全地访问虚拟机

具体地说，Azure Bastion 直接在 Azure 门户中为 Azure 虚拟网络中的虚拟机提供安全无缝的 RDP/SSH 连接，而无需使用公共 IP 地址。 若要详细了解 Azure Bastion 体系结构和主要功能，请参阅[什么是 Azure Bastion](bastion-overview.md)。

Azure Bastion 按虚拟网络部署，这意味着公司可以配置和管理一个 Azure Bastion，以快速支持远程用户访问 Azure 虚拟网络中的虚拟机。 有关如何创建和管理 Azure Bastion 的指导，请参阅[创建堡垒主机](./tutorial-create-host-portal.md)。

## <a name="next-steps"></a>后续步骤

* 使用 [Azure 门户](./tutorial-create-host-portal.md)、[PowerShell](bastion-create-host-powershell.md) 或 Azure CLI 配置 Azure Bastion。

* 有关其他信息，请参阅 [Bastion 常见问题解答](bastion-faq.md)。

<!-- Update_Description: update meta properties, wording update, update link -->