---
title: 筛选 VM 网络流量 - Azure CLI 脚本示例
description: 使用 Azure CLI 脚本示例筛选入站和出站虚拟机 (VM) 网络流量。
services: virtual-network
documentationcenter: virtual-network
manager: mtillman
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
origin.date: 03/20/2018
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: yes
ms.testdate: 08/10/2020
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3bc639734d38f3b388a0fa624f2799fc7fbecce6
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564522"
---
# <a name="filter-inbound-and-outbound-vm-network-traffic-using-an-azure-cli-script-sample"></a>使用 Azure CLI 脚本示例筛选入站和出站 VM 网络流量

该脚本示例创建了包含前端和后端子网的虚拟网络。 前端子网的入站网络流量仅限于 HTTP、HTTPS 和 SSH，而从后端子网到 Internet 的出站流量则不受允许。 运行该脚本后，将具有一个包含两个 NIC 的虚拟机。 每个 NIC 连接到不同的子网。

可以通过本地 Azure CLI 安装来执行脚本。 如果在本地使用 CLI，此脚本要求运行版本 2.0.28 或更高版本。 要查找已安装的版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。 如果在本地运行 CLI，则还需运行 `az login` 以创建与 Azure 的连接。

<!-- Not Available on [Cloud Shell](https://shell.azure.com/bash) -->

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/bash

RgName="MyResourceGroup"
Location="chinaeast"

# Create a resource group.
az group create \
  --name $RgName \
  --location $Location

# Create a virtual network and a front-end subnet.
az network vnet create \
  --resource-group $RgName \
  --name MyVnet \
  --address-prefix 10.0.0.0/16  \
  --location $Location \
  --subnet-name MySubnet-FrontEnd \
  --subnet-prefix 10.0.1.0/24

# Create a back-end subnet.
az network vnet subnet create \
  --address-prefix 10.0.2.0/24 \
  --name MySubnet-BackEnd \
  --resource-group $RgName \
  --vnet-name MyVnet

# Create a network security group (NSG) for the front-end subnet.
az network nsg create \
  --resource-group $RgName \
  --name MyNsg-FrontEnd \
  --location $Location

# Create NSG rules to allow HTTP & HTTPS traffic inbound.
az network nsg rule create \
  --resource-group $RgName \
  --nsg-name MyNsg-FrontEnd \
  --name Allow-HTTP-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 80

az network nsg rule create \
  --resource-group $RgName \
  --nsg-name MyNsg-FrontEnd \
  --name Allow-HTTPS-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 200 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 443

# Create an NSG rule to allow SSH traffic in from the Internet to the front-end subnet.
az network nsg rule create \
  --resource-group $RgName \
  --nsg-name MyNsg-FrontEnd \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 300 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 22

# Associate the front-end NSG to the front-end subnet.
az network vnet subnet update \
  --vnet-name MyVnet \
  --name MySubnet-FrontEnd \
  --resource-group $RgName \
  --network-security-group MyNsg-FrontEnd

# Create a network security group for the back-end subnet.
az network nsg create \
  --resource-group $RgName \
  --name MyNsg-BackEnd \
  --location $Location

# Create an NSG rule to block all outbound traffic from the back-end subnet to the Internet (inbound blocked by default).
az network nsg rule create \
  --resource-group $RgName \
  --nsg-name MyNsg-BackEnd \
  --name Deny-Internet-All \
  --access Deny --protocol Tcp \
  --direction Outbound --priority 100 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "Internet" \
  --destination-port-range "*"

# Associate the back-end NSG to the back-end subnet.
az network vnet subnet update \
  --vnet-name MyVnet \
  --name MySubnet-BackEnd \
  --resource-group $RgName \
  --network-security-group MyNsg-BackEnd

# Create a public IP address for the VM front-end network interface.
az network public-ip create \
  --resource-group $RgName \
  --name MyPublicIp-FrontEnd \
  --allocation-method Dynamic

# Create a network interface for the VM attached to the front-end subnet.
az network nic create \
  --resource-group $RgName \
  --vnet-name MyVnet \
  --subnet MySubnet-FrontEnd \
  --name MyNic-FrontEnd \
  --public-ip-address MyPublicIp-FrontEnd

# Create a network interface for the VM attached to the back-end subnet.
az network nic create \
  --resource-group $RgName \
  --vnet-name MyVnet \
  --subnet MySubnet-BackEnd \
  --name MyNic-BackEnd

# Create the VM with both the FrontEnd and BackEnd NICs.
az vm create \
  --resource-group $RgName \
  --name MyVm \
  --nics MyNic-FrontEnd MyNic-BackEnd \
  --image UbuntuLTS \
  --admin-username azureadmin \
  --generate-ssh-keys

```

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源：

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟网络和网络安全组。 下表中的每条命令均链接到特定于命令的文档：

| Command | 说明 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](https://docs.azure.cn/cli/network/vnet#az-network-vnet-create) | 创建 Azure 虚拟网络和前端子网。 |
| [az network subnet create](https://docs.azure.cn/cli/network/vnet/subnet#az-network-subnet-create) | 创建后端子网。 |
| [az network vnet subnet update](https://docs.azure.cn/cli/network/vnet/subnet#az-network-vnet-subnet-update) | 将 NSG 关联到子网。 |
| [az network public-ip create](https://docs.azure.cn/cli/network/public-ip#az-network-public-ip-create) | 创建用于从 Internet 访问 VM 的公共 IP 地址。 |
| [az network nic create](https://docs.azure.cn/cli/network/nic#az-network-nic-create) | 创建虚拟网络接口，并将其附加到虚拟网络的前端和后端子网。 |
| [az network nsg create](https://docs.azure.cn/cli/network/nsg#az-network-nsg-create) | 创建关联到前端和后端子网的网络安全组 (NSG)。 |
| [az network nsg rule create](https://docs.azure.cn/cli/network/nsg/rule#az-network-nsg-rule-create) |创建 NSG 规则，允许或阻止特定子网的特定端口。 |
| [az vm create](https://docs.azure.cn/cli/vm#az-vm-create) | 创建虚拟机，并将 NIC 附加到每个 VM。 此命令还指定要使用的虚拟机映像和管理凭据。 |
| [az group delete](https://docs.azure.cn/cli/group#az-group-delete) | 删除资源组及其包含的所有资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.azure.cn/cli/index)。

可在[虚拟网络 CLI 示例](../cli-samples.md)中查找其他虚拟网络 CLI 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->