---
title: 适用于虚拟网络的 Azure CLI 示例
description: 了解可用于在 Azure CLI 中完成任务的各种示例脚本，包括为多层应用程序创建虚拟网络。
services: virtual-network
documentationcenter: virtual-network
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: sample
ms.workload: infrastructure
origin.date: 07/15/2019
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: dad2e17a090cc554190c4da34583d4a6f607d13d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106055"
---
# <a name="azure-cli-samples-for-virtual-network"></a>适用于虚拟网络的 Azure CLI 示例

下表包含指向使用 Azure CLI 命令的 bash 脚本的链接：

| Script | 说明 |
|----|----|
| [为多层应用程序创建虚拟网络](./scripts/virtual-network-cli-sample-multi-tier-application.md) | 创建包含前端和后端子网的虚拟网络。 传入前端子网的流量仅限 HTTP 和 SSH，而传入后端子网的流量限于 MySQL、端口 3306。 |
| [两个对等虚拟网络](./scripts/virtual-network-cli-sample-peer-two-virtual-networks.md) | 在同一区域中创建并连接两个虚拟网络。 |
| [通过网络虚拟设备的路由流量](./scripts/virtual-network-cli-sample-route-traffic-through-nva.md) | 创建包含前端和后端子网的虚拟网络，以及可在这两个子网之间路由流量的 VM。 |
| [筛选入站和出站 VM 网络流量](./scripts/virtual-network-cli-sample-filter-network-traffic.md) | 创建包含前端和后端子网的虚拟网络。 前端子网的入站网络流量仅限于 HTTP、HTTPS 和 SSH。 不允许从后端子网到 Internet 的出站流量。 |
|[使用基本负载均衡器配置 IPv4 + IPv6 双堆栈虚拟网络](./scripts/virtual-network-cli-sample-ipv6-dual-stack.md)|部署具有两个 VM 的双栈 (IPv4+IPv6) 虚拟网络和具有 IPv4 和 IPv6 公共 IP 地址的 Azure 基本负载均衡器。 |
|[使用标准负载均衡器配置 IPv4 + IPv6 双堆栈虚拟网络](./scripts/virtual-network-cli-sample-ipv6-dual-stack-standard-load-balancer.md)|部署具有两个 VM 的双堆栈 (IPv4+IPv6) 虚拟网络和具有 IPv4 和 IPv6 公共 IP 地址的 Azure 标准负载均衡器。 |

<!--Not Available on [Tutorial: Create and test a NAT gateway - Azure CLI](../virtual-network/tutorial-create-validate-nat-gateway-cli.md)-->

<!-- Update_Description: update meta properties, wording update, update link -->