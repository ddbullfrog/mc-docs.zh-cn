---
title: 将 IPv6 添加到 Azure 虚拟网络中的 IPv4 应用程序 - Azure CLI
titlesuffix: Azure Virtual Network
description: 本文介绍如何使用 Azure CLI 将 IPv6 地址部署到 Azure 虚拟网络中的现有应用程序。
services: virtual-network
documentationcenter: na
manager: digimobile
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 03/31/2020
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: yes
ms.testdate: 10/05/2020
ms.author: v-yeche
ms.openlocfilehash: 3f38c6df3977958bb766211a98774aa3b5b51d6b
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564459"
---
# <a name="add-ipv6-to-an-ipv4-application-in-azure-virtual-network---azure-cli"></a>将 IPv6 添加到 Azure 虚拟网络中的 IPv4 应用程序 - Azure CLI

<!--MOONCAKE: REMOVE preview tag-->

本文介绍如何通过 Azure CLI，将 IPv6 地址添加到对标准负载均衡器使用 Azure 虚拟网络中 IPv4 公共 IP 地址的应用程序。 就地升级涉及到虚拟网络和子网、采用 IPv4 + IPV6 前端配置的标准负载均衡器、包含采用 IPv4 + IPv6 配置的 NIC 的 VM、网络安全组和公共 IP。

<!--MOONCAKE: REMOVE preview tag of [!Important]-->

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

本快速入门需要使用 Azure CLI 版本 2.0.28 或更高版本。 若要查找已安装的版本，请运行 `az --version`。 有关安装或升级信息，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

<!--Not Available on If you decide to install and use Azure CLI locally instead,-->

## <a name="prerequisites"></a>先决条件

本文假设已根据以下文章所述部署了一个标准负载均衡器：[快速入门：创建标准负载均衡器 - Azure CLI](../load-balancer/quickstart-load-balancer-standard-public-cli.md)。

## <a name="create-ipv6-addresses"></a>创建 IPv6 地址

使用 [az network public-ip create](https://docs.azure.cn/cli/network/public-ip#az-network-public-ip-create) 为标准负载均衡器创建公共 IPv6 地址。 以下示例在 *myResourceGroupSLB* 资源组中创建名为 *PublicIP_v6* 的 IPv6 公共 IP 地址：

```azurecli
az network public-ip create \
--name PublicIP_v6 \
--resource-group MyResourceGroupSLB \
--location ChinaEast \
--sku Standard \
--allocation-method static \
--version IPv6
```

## <a name="configure-ipv6-load-balancer-frontend"></a>配置 IPv6 负载均衡器前端

运行 [az network lb frontend-ip create](https://docs.azure.cn/cli/network/lb/frontend-ip#az-network-lb-frontend-ip-create) 配置使用新 IPv6 IP 地址的负载均衡器，如下所示：

```azurecli
az network lb frontend-ip create \
--lb-name myLoadBalancer \
--name dsLbFrontEnd_v6 \
--resource-group MyResourceGroupSLB \
--public-ip-address PublicIP_v6
```

## <a name="configure-ipv6-load-balancer-backend-pool"></a>配置 IPv6 负载均衡器后端池

运行 [az network lb address-pool create](https://docs.azure.cn/cli/network/lb/address-pool#az-network-lb-address-pool-create) 为使用 IPv6 地址的 NIC 创建后端池，如下所示：

```azurecli
az network lb address-pool create \
--lb-name myLoadBalancer \
--name dsLbBackEndPool_v6 \
--resource-group MyResourceGroupSLB
```

## <a name="configure-ipv6-load-balancer-rules"></a>配置 IPv6 负载均衡器规则

使用 [az network lb rule create](https://docs.azure.cn/cli/network/lb/rule#az-network-lb-rule-create) 创建 IPv6 负载均衡器规则。

```azurecli
az network lb rule create \
--lb-name myLoadBalancer \
--name dsLBrule_v6 \
--resource-group MyResourceGroupSLB \
--frontend-ip-name dsLbFrontEnd_v6 \
--protocol Tcp \
--frontend-port 80 \
--backend-port 80 \
--backend-pool-name dsLbBackEndPool_v6
```

## <a name="add-ipv6-address-ranges"></a>添加 IPv6 地址范围

将 IPv6 地址范围添加到托管负载均衡器的虚拟网络和子网，如下所示：

```azurecli
az network vnet update \
--name myVnet  \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/16"  "ace:cab:deca::/48"

az network vnet subnet update \
--vnet-name myVnet \
--name mySubnet \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/24"  "ace:cab:deca:deed::/64"  
```

## <a name="add-ipv6-configuration-to-nics"></a>将 IPv6 配置添加到 NIC

运行 [az network nic ip-config create](https://docs.azure.cn/cli/network/nic/ip-config#az-network-nic-ip-config-create) 配置使用 IPv6 地址的 VM NIC，如下所示：

```azurecli
az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name myNicVM1 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC2 \
--nic-name myNicVM2 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer

az network nic ip-config create \
--name dsIp6Config_NIC3 \
--nic-name myNicVM3 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>在 Azure 门户中查看 IPv6 双堆栈虚拟网络

可以在 Azure 门户中查看 IPv6 双堆栈虚拟网络，如下所示：
1. 在门户的搜索栏中输入 *myVnet*。
2. 当“myVnet”出现在搜索结果中时，将其选中。**** 此时会启动名为 *myVNet* 的双堆栈虚拟网络的“概述”页。**** 该双堆栈虚拟网络显示了位于 *mySubnet* 双堆栈子网中的三个 NIC，这些 NIC 采用 IPv4 和 IPv6 配置。

    :::image type="content" source="./media/ipv6-add-to-existing-vnet-powershell/ipv6-dual-stack-vnet.png" alt-text="Azure 中的 IPv6 双堆栈虚拟网络":::

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、VM 和所有相关的资源，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 命令将其删除。

```azurecli
az group delete --name MyAzureResourceGroupSLB
```

## <a name="next-steps"></a>后续步骤

在本文中，你已将采用 IPv4 前端 IP 配置的现有标准负载均衡器更新为双堆栈（IPv4 和 IPv6）配置。 你还将 IPv6 配置添加到了后端池中 VM 的 NIC。 若要详细了解 Azure 虚拟网络中的 IPv6 支持，请参阅 [Azure 虚拟网络 IPv6 是什么？](ipv6-overview.md)

<!-- Update_Description: update meta properties, wording update, update link -->