---
title: 在虚拟 WAN 中心安装 Azure 防火墙
titleSuffix: Azure Virtual WAN
description: 在虚拟 WAN 中心配置 Azure 防火墙的步骤
services: virtual-wan
ms.service: virtual-wan
ms.topic: how-to
origin.date: 09/22/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.openlocfilehash: 7803ffaf02cd9a6140485b1dcd23295082bcc935
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92471266"
---
# <a name="configure-azure-firewall-in-a-virtual-wan-hub"></a>在虚拟 WAN 中心配置 Azure 防火墙

安全中心是使用 Azure 防火墙的 Azure 虚拟 WAN 中心。 本文将指导你完成通过直接从 Azure 虚拟 WAN 门户页面安装 Azure 防火墙将虚拟 WAN 中心转换为安全中心的步骤。

## <a name="before-you-begin"></a>开始之前

本文中的步骤假设你已部署了具有一个或多个中心的虚拟 WAN。

若要创建新虚拟 WAN 和新中心，请使用以下文章中的步骤：

* [创建虚拟 WAN](virtual-wan-site-to-site-portal.md#openvwan)
* [创建中心](virtual-wan-site-to-site-portal.md#hub)

## <a name="view-virtual-hubs"></a>查看虚拟中心

虚拟 WAN 的“概述”页面显示了虚拟中心和安全中心的列表。 下图显示没有安全中心的虚拟 WAN。

[ :::image type="content" source="./media/howto-firewall/overview.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/overview.png#lightbox)

## <a name="convert-to-secured-hub"></a>转换为安全中心

1. 在虚拟 WAN 的“概述”页面上，选择要转换为安全中心的中心。 在虚拟中心页面上，可看到两个选项，用于将 Azure 防火墙部署到此中心。 选择任一选项。

    [ :::image type="content" source="./media/howto-firewall/security.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/security.png#lightbox)

1. 选择其中一个选项之后，会看到“转换为安全中心”页面。 选择要转换的中心，然后选择页面底部的“下一步:Azure 防火墙”。

    [ :::image type="content" source="./media/howto-firewall/select-hub.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/select-hub.png#lightbox)
1. 完成工作流之后，选择“确认”。

    [ :::image type="content" source="./media/howto-firewall/confirm.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/confirm.png#lightbox)

1. 将中心转换为安全中心之后，可在虚拟 WAN“概述”页面上查看它。

    [ :::image type="content" source="./media/howto-firewall/secured-hub.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/secured-hub.png#lightbox)

## <a name="view-hub-resources"></a>查看中心资源

从虚拟 WAN“概述”页面中，选择安全中心。 在中心页面上，可以查看所有虚拟中心资源，包括 Azure 防火墙。

[ :::image type="content" source="./media/howto-firewall/view-resources.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/view-resources.png#lightbox)

若要从安全中心查看 Azure 防火墙设置，请在“安全性”下，选择“安全虚拟中心设置”。
[ :::image type="content" source="./media/howto-firewall/hub-settings.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/hub-settings.png#lightbox)

## <a name="configure-additional-settings"></a>配置其他设置

若要为虚拟中心配置其他 Azure 防火墙设置，请选择“Azure 防火墙管理器”的链接。 

<!--Not Available on [Azure Firewall Manager](../firewall-manager/secure-cloud-network.md#create-a-firewall-policy-and-secure-your-hub)-->

[ :::image type="content" source="./media/howto-firewall/additional-settings.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/additional-settings.png#lightbox)

若要返回到中心“概述”页面，可以通过单击路径向后导航，如下图中的箭头所示。

[ :::image type="content" source="./media/howto-firewall/arrow.png" alt-text="屏幕截图显示了虚拟 WAN 的“概览”页，其中包含虚拟中心列表。":::](./media/howto-firewall/arrow.png#lightbox)

## <a name="next-steps"></a>后续步骤

有关虚拟 WAN 的详细信息，请参阅[常见问题解答](virtual-wan-faq.md)。

<!-- Update_Description: update meta properties, wording update, update link -->