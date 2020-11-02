---
title: 教程 - 使用 Azure 虚拟 WAN 与 Azure 建立点到站点连接
description: 本教程介绍如何使用 Azure 虚拟 WAN 与 Azure 建立点到站点 VPN 连接。
services: virtual-wan
ms.service: virtual-wan
ms.topic: tutorial
author: rockboyfor
origin.date: 10/06/2020
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.openlocfilehash: be370a236f7e1cf9a06abd792fb40c9c769ea18c
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472541"
---
# <a name="tutorial-create-a-user-vpn-connection-using-azure-virtual-wan"></a>教程：使用 Azure 虚拟 WAN 创建用户 VPN 连接

本教程介绍如何使用虚拟 WAN 通过 IPsec/IKE (IKEv2) 或 OpenVPN VPN 连接与 Azure 中的资源建立连接。 此类连接要求在客户端计算机上配置一个客户端。 有关虚拟 WAN 的详细信息，请参阅[虚拟 WAN 概述](virtual-wan-about.md)

在本教程中，你将了解如何执行以下操作：

> [!div class="checklist"]
> * 创建 WAN
> * 创建 P2S 配置
> * 创建中心
> * 指定 DNS 服务器
> * 下载 VPN 客户端配置文件
> * 查看虚拟 WAN

:::image type="content" source="./media/virtual-wan-about/virtualwanp2s.png" alt-text="虚拟 WAN 示意图":::

## <a name="prerequisites"></a>先决条件

[!INCLUDE [Before beginning](../../includes/virtual-wan-before-include.md)]

<a name="wan"></a>
## <a name="create-a-virtual-wan"></a>创建虚拟 WAN

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

<a name="p2sconfig"></a>
## <a name="create-a-p2s-configuration"></a>创建 P2S 配置

点到站点 (P2S) 配置定义连接远程客户端的参数。

[!INCLUDE [Create client profiles](../../includes/virtual-wan-p2s-configuration-include.md)]

<a name="hub"></a>
## <a name="create-hub-with-point-to-site-gateway"></a>使用点到站点网关创建中心

[!INCLUDE [Create hub](../../includes/virtual-wan-p2s-hub-include.md)]

<a name="dns"></a>
## <a name="specify-dns-server"></a>指定 DNS 服务器

虚拟 WAN 用户 VPN 网关允许指定最多 5 个 DNS 服务器。 可以在创建中心的过程中对其进行配置，也可以在以后对其进行修改。 若要执行此操作，请找到虚拟中心。 在“用户 VPN(点到站点)”下，选择“配置”，然后在“自定义 DNS 服务器”文本框中输入 DNS 服务器 IP 地址  。

:::image type="content" source="media/virtual-wan-point-to-site-portal/custom-dns.png" alt-text="虚拟 WAN 示意图" lightbox="media/virtual-wan-point-to-site-portal/custom-dns-expand.png":::

<a name="download"></a>
## <a name="download-vpn-profile"></a>下载 VPN 配置文件

使用 VPN 配置文件来配置客户端。

[!INCLUDE [Download profile](../../includes/virtual-wan-p2s-download-profile-include.md)]

### <a name="configure-user-vpn-clients"></a>配置用户 VPN 客户端

使用下载的配置文件配置远程访问客户端。 每个操作系统的过程均不同，请按照适用于你的系统的说明进行操作。

[!INCLUDE [Configure clients](../../includes/virtual-wan-p2s-configure-clients-include.md)]

<a name="viewwan"></a>
## <a name="view-your-virtual-wan"></a>查看虚拟 WAN

1. 导航到虚拟 WAN。
1. 在“概述”页上，地图中的每个点表示一个中心。
1. 在“中心和连接”部分，可以查看中心状态、站点、区域、VPN 连接状态和传入与传出字节数。

<a name="cleanup"></a>
## <a name="clean-up-resources"></a>清理资源

如果不再需要这些资源，可以使用 [Remove-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/remove-azurermresourcegroup) 删除资源组及其包含的所有资源。 将“myResourceGroup”替换为资源组的名称，并运行以下 PowerShell 命令：

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>后续步骤

接下来，若要详细了解虚拟 WAN，请参阅：

> [!div class="nextstepaction"]
> * [虚拟 WAN 常见问题解答](virtual-wan-faq.md)


<!-- Update_Description: update meta properties, wording update, update link -->