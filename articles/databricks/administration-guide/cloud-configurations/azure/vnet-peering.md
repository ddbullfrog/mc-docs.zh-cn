---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 将虚拟网络对等互连 - Azure Databricks
description: 了解如何配置虚拟网络对等互连。
ms.openlocfilehash: ee9ad086b020ef5a2d7f590ffba763ce1fa475cb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106516"
---
# <a name="peer-virtual-networks"></a><a id="peer-virtual-networks"> </a><a id="vnet-peering"> </a>将虚拟网络对等互连

通过虚拟网络 (VNet) 对等互连，正在运行 Azure Databricks 资源的虚拟网络可与另一个 Azure 虚拟网络对等互连。 对等虚拟网络中虚拟机之间的流量通过 Microsoft 主干基础结构路由，非常类似于只通过专用 IP 地址在同一虚拟网络中的虚拟机之间路由流量。 有关 Azure VNet 对等互连的概述，请参阅 [Microsoft Azure 虚拟网络对等互连](/virtual-network/virtual-network-peering-overview)。

本文介绍如何将 Azure Databricks VNet 与 Azure VNet 对等互连，并提供有关如何将本地 VNet 连接到 Azure VNet 的参考信息。

有关如何管理 Azure VNet 对等互连的信息，请参阅[创建、更改或删除虚拟网络对等互连](/virtual-network/virtual-network-manage-peering)。 但是，请不要按照该文中的步骤在 Azure Databricks VNet 和 Azure VNet 之间创建 VNet 对等互连；请按本文中提供的说明进行操作。

## <a name="peer-an-azure-databricks-virtual-network-to-a-remote-virtual-network"></a>将 Azure Databricks 虚拟网络对等互连到远程虚拟网络

将 Databricks 虚拟网络对等互连到 Azure VNet 涉及两个步骤：将远程虚拟网络对等互连添加到 Azure Databricks 虚拟网络，并将 Azure Databricks 虚拟网络对等互连添加到远程虚拟网络。

### <a name="step-1-add-remote-virtual-network-peering-to-azure-databricks-virtual-network"></a>步骤 1：将远程虚拟网络对等互连添加到 Azure Databricks 虚拟网络

1. 在 Azure 门户中，单击 Azure Databricks 服务资源。
2. 在边栏的“设置”部分中，单击“虚拟网络对等互连”选项卡。
3. 单击“+ 添加对等互连”按钮。

   > [!div class="mx-imgBorder"]
   > ![添加对等互连按钮](../../../_static/images/vpc-peer/azure-virtual-network-peering1.png)

4. 在“名称”字段中，输入对等互连虚拟网络的名称。
5. 根据你对远程虚拟网络了解的信息，请执行以下操作之一：
   * 你知道远程虚拟网络的资源 ID：
     1. 选择“我知道我的资源 ID”复选框。
     1. 在“资源 ID”文本框中，粘贴远程虚拟网络资源 ID。

        > [!div class="mx-imgBorder"]
        > ![添加远程虚拟网络资源 ID](../../../_static/images/vpc-peer/azure-add-peering1a.png)

   * 你知道远程虚拟网络的名称：
     1. 在“订阅”下拉列表中，选择一个订阅。
     1. 在“虚拟网络”下拉列表中，选择远程虚拟网络。

        > [!div class="mx-imgBorder"]
        > ![选择远程虚拟网络](../../../_static/images/vpc-peer/azure-add-peering1b.png)

6. 在“配置”部分，指定对等互连的配置。 参阅[创建对等互连](/virtual-network/virtual-network-manage-peering#create-a-peering)，获取配置字段的相关信息。
7. 在“Databricks 虚拟网络资源 Id”部分中，复制“资源 ID”。
8. 单击 **添加** 。 已部署虚拟网络对等互连。

### <a name="step-2-add-azure-databricks-virtual-network-peer-to-remote-virtual-network"></a>步骤 2：将 Azure Databricks 虚拟网络对等互连添加到远程虚拟网络

1. 在 Azure 门户边栏中，单击“虚拟网络”。
2. 搜索要与 Databricks 虚拟网络对等互连的虚拟网络资源，然后单击资源名称。
3. 在边栏的“设置”部分中，单击“对等互连”选项卡。
4. 单击“+ 添加”按钮。

   > [!div class="mx-imgBorder"]
   > ![添加对等互连](../../../_static/images/vpc-peer/azure-virtual-network-peering2.png)

5. 在“名称”字段，输入对等互连虚拟网络的名称。
6. 对等互连的详细信息中，“虚拟网络部署模型”必须是“资源管理器” 。
7. 选择“我知道我的资源 ID”复选框。
8. 在“资源 ID”文本框中，粘贴在步骤 1 中复制的 Databricks 虚拟网络资源 ID。

   > [!div class="mx-imgBorder"]
   > ![添加对等互连网络 ID](../../../_static/images/vpc-peer/azure-add-peering2.png)

9. 在“配置”部分，指定对等互连的配置。 参阅[创建对等互连](/virtual-network/virtual-network-manage-peering#create-a-peering)，获取配置字段的相关信息。
10. 单击“确定”。 已部署虚拟网络对等互连。

## <a name="connect-an-on-premises-virtual-network-to-an-azure-virtual-network"></a>将本地虚拟网络连接到 Azure 虚拟网络

若要将本地网络连接到 Azure VNet，请按照[使用 ExpressRoute 将本地网络连接到 Azure](/architecture/reference-architectures/hybrid-networking/expressroute) 中的步骤进行操作。

若要创建从本地网络到 Azure VNet 的站点到站点 VPN 网关连接，请按照[在 Azure 门户中创建站点到站点连接](/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal)中的步骤进行操作。