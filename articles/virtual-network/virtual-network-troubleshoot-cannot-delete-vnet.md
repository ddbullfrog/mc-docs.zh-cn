---
title: 无法在 Azure 中删除虚拟网络 | Azure
description: 了解如何排查无法在 Azure 中删除虚拟网络的问题。
services: virtual-network
documentationcenter: na
manager: digimobile
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 10/31/2018
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 0d4bd072d680efcea3da11b044c90cf67fcf1622
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564511"
---
# <a name="troubleshooting-failed-to-delete-a-virtual-network-in-azure"></a>故障排除：无法在 Azure 中删除虚拟网络

尝试在 Azure 中删除虚拟网络时，可能会收到错误。 本文提供解决此问题的故障排除步骤。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-guidance"></a>故障排除指南 

1. [检查虚拟网络网关是否在虚拟网络中运行](#check-whether-a-virtual-network-gateway-is-running-in-the-virtual-network)。
2. [检查应用程序网关是否在虚拟网络中运行](#check-whether-an-application-gateway-is-running-in-the-virtual-network)。
3. [检查 Azure Active Directory 域服务是否已在虚拟网络中启用](#check-whether-azure-active-directory-domain-service-is-enabled-in-the-virtual-network)。
4. [检查虚拟网络是否已连接到其他资源](#check-whether-the-virtual-network-is-connected-to-other-resource)。
5. [检查虚拟机是否仍在虚拟网络中运行](#check-whether-a-virtual-machine-is-still-running-in-the-virtual-network)。
6. [检查虚拟网络是否停滞在迁移状态](#check-whether-the-virtual-network-is-stuck-in-migration)。

## <a name="troubleshooting-steps"></a>疑难解答步骤

### <a name="check-whether-a-virtual-network-gateway-is-running-in-the-virtual-network"></a>检查虚拟网络网关是否在虚拟网络中运行

若要删除虚拟网络，首先必须删除虚拟网络网关。

对于经典虚拟网络，请在 Azure 门户中转到经典虚拟网络的“概述”页。 在“VPN 连接”部分中，如果网关正在虚拟网络中运行，则会显示该网关的 IP 地址。 

:::image type="content" source="media/virtual-network-troubleshoot-cannot-delete-vnet/classic-gateway.png" alt-text="检查网关是否正在运行":::

对于虚拟网络，请转到虚拟网络的“概述”页。 检查虚拟网络网关的“已连接设备”。

:::image type="content" source="media/virtual-network-troubleshoot-cannot-delete-vnet/vnet-gateway.png" alt-text="检查网关是否正在运行":::

在删除网关之前，请先删除该网关中的所有“连接”对象。 

### <a name="check-whether-an-application-gateway-is-running-in-the-virtual-network"></a>检查应用程序网关是否在虚拟网络中运行

转到虚拟网络的“概述”页。 检查应用程序网关的“已连接设备”。

:::image type="content" source="media/virtual-network-troubleshoot-cannot-delete-vnet/app-gateway.png" alt-text="检查网关是否正在运行":::

如果存在应用程序网关，则必须先将其删除，然后才能删除虚拟网络。

### <a name="check-whether-azure-active-directory-domain-service-is-enabled-in-the-virtual-network"></a>检查 Azure Active Directory 域服务是否已在虚拟网络中启用

如果 Active Directory 域服务已启用并已连接到虚拟网络，则无法删除此虚拟网络。 

:::image type="content" source="media/virtual-network-troubleshoot-cannot-delete-vnet/enable-domain-services.png" alt-text="检查网关是否正在运行":::

<!-- Not Available on  [Disable Azure Active Directory Domain Services using the Azure portal](../active-directory-domain-services/active-directory-ds-disable-aadds.md). -->

### <a name="check-whether-the-virtual-network-is-connected-to-other-resource"></a>检查虚拟网络是否已连接到其他资源

检查线路链接、连接和虚拟网络对等互连。 其中的任何对象都可能导致虚拟网络删除失败。 

建议的删除顺序如下：

1. 网关连接
2. 网关
3. IP
4. 虚拟网络对等互连
5. 应用服务环境 (ASE)

### <a name="check-whether-a-virtual-machine-is-still-running-in-the-virtual-network"></a>检查虚拟机是否仍在虚拟网络中运行

确保虚拟网络中没有任何虚拟机。

### <a name="check-whether-the-virtual-network-is-stuck-in-migration"></a>检查虚拟网络是否停滞在迁移状态

如果虚拟网络停滞在迁移状态，则无法将其删除。 请运行以下命令中止迁移，然后删除虚拟网络。

```azurepowershell
Move-AzureVirtualNetwork -VirtualNetworkName "Name" -Abort
```

## <a name="next-steps"></a>后续步骤

- [Azure 虚拟网络](virtual-networks-overview.md)
- [Azure 虚拟网络常见问题解答 (FAQ)](virtual-networks-faq.md)

<!-- Update_Description: update meta properties, wording update, update link -->