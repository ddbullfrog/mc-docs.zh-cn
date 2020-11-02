---
title: include 文件
description: include 文件
services: virtual-network
ms.service: virtual-network
ms.topic: include
origin.date: 04/09/2018
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 1a55be75853bc99b8aeeede256792a9904cffde9
ms.sourcegitcommit: 1f933e4790b799ceedc685a0cea80b1f1c595f3d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92628253"
---
> [!div class="op_single_selector"]
> * [Azure 门户](../articles/virtual-network/virtual-network-multiple-ip-addresses-portal.md)
> * [PowerShell](../articles/virtual-network/virtual-network-multiple-ip-addresses-powershell.md)
> * [Azure CLI](../articles/virtual-network/virtual-network-multiple-ip-addresses-cli.md)
>

在一个 Azure 虚拟机 (VM) 上可以附加一个或多个网络接口 (NIC)。 可为任何 NIC 分配一个或多个静态或动态的公共与专用 IP 地址。 为 VM 分配多个 IP 地址可实现以下功能：

* 在单个服务器上使用不同的 IP 地址和 SSL 证书托管多个网站或服务。
* 用作网络虚拟设备，例如防火墙或负载均衡器。
* 可将任何 NIC 的任何专用 IP 地址添加到 Azure 负载均衡器后端池。 以往，只能将主要 NIC 的主要 IP 地址添加到后端池。 若要详细了解如何对多个 IP 配置进行负载均衡，请阅读[对多个 IP 配置进行负载均衡](../articles/load-balancer/load-balancer-multiple-ip.md?toc=%2fvirtual-network%2ftoc.json)一文。

附加到 VM 的每个 NIC 都具有一个或多个关联的 IP 配置。 每个配置分配有一个静态或动态专用 IP 地址。 每个配置还可以具有一个关联的公共 IP 地址资源。 公共 IP 地址资源分配有动态或静态公共 IP 地址。 若要详细了解 Azure 中的 IP 地址，请阅读 [Azure 中的 IP 地址](../articles/virtual-network/virtual-network-ip-addresses-overview-arm.md)一文。 

分配给 NIC 的专用 IP 地址数目存在限制。 能够在 Azure 订阅中使用的公共 IP 地址数也存在限制。 有关详细信息，请参阅 [Azure 限制](../articles/azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)一文。

<!-- Update_Description: update meta properties, wording update, update link -->