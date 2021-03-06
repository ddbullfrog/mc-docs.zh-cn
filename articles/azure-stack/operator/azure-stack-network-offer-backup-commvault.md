---
title: 将 Commvault 添加到 Azure Stack Hub 市场
description: 了解如何将 Commvault 添加到 Azure Stack Hub 市场。
author: WenJason
ms.topic: article
origin.date: 04/20/2020
ms.date: 06/22/2020
ms.author: v-jay
ms.reviewer: sijuman
ms.lastreviewed: 10/28/2019
ms.openlocfilehash: 03c97ab07b084f81cccfcc71854c622d29432010
ms.sourcegitcommit: d86e169edf5affd28a1c1a4476d72b01a7fb421d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85096509"
---
# <a name="add-commvault-to-azure-stack-hub-marketplace"></a>将 Commvault 添加到 Azure Stack Hub 市场

本文逐步讲解如何提供 Commvault Live Sync，以更新位于独立 Azure Stack Hub 缩放单元上的恢复虚拟机 (VM)。 可以下载 Commvault 并将其作为用户的备份和复制解决方案提供。

## <a name="notes-for-commvault"></a>Commvault 说明

- 用户需在其源 Azure Stack Hub 订阅中的 VM 上安装备份和复制软件。 Azure Site Recovery 和 Azure 备份可以提供一个出栈位置来存储备份和恢复映像。 它们都要求先在 Azure 中创建恢复服务保管库，然后再下载要安装到 Azure Stack Hub 上的软件映像。 可以从以下位置下载软件映像：[Azure 备份服务器](https://go.microsoft.com/fwLink/?LinkId=626082&clcid=0x0409)和 [Azure Site Recovery](https://aka.ms/unifiedinstaller_eus)。  

- 可能需要第三方软件（如果已选择）的许可证。
- 用户可能需要帮助才能通过位于备份和复制主机上的 VPN 网关或公共 IP 连接其源和目标。
- 以 Azure 云订阅或恢复目标 Azure Stack Hub 上的订阅为目标。
- 以恢复目标 Azure Stack Hub 上的资源组和 Blob 存储帐户为目标。
- 某些解决方案要求在目标订阅中创建需要 365 天全天候运行的 VM 才能从源服务器接收更改。 在[使用 Commvault 在 Azure Stack Hub 上备份 VM](../user/azure-stack-network-howto-backup-commvault.md) 中，Commvault Live Sync 在初始配置过程中创建目标恢复 VM，并让其一直处于空闲（不运行、不计费）状态，直到需要在复制周期应用更改为止。

## <a name="get-commvault-for-your-marketplace"></a>获取用于市场的 Commvault

1. 打开 Azure Stack Hub 管理员门户。
2. 选择“市场管理” > “从 Azure 添加”。 

    ![用于 Azure Stack Hub 的 Commvault](./media/azure-stack-network-offer-backup-commvault/get-commvault-for-marketplace.png)

3. 输入 `commvault`。
4. 选择“Commvault 试用版”。 然后选择“下载”。

## <a name="next-steps"></a>后续步骤

- [使用 Commvault 在 Azure Stack Hub 上备份 VM](../user/azure-stack-network-howto-backup-commvault.md)
- [在 Azure Stack Hub 中提供服务概述](service-plan-offer-subscription-overview.md)
