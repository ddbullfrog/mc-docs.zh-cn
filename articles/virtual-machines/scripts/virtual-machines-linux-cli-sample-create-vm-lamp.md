---
title: Azure CLI 脚本示例 - 在负载均衡虚拟机规模集中部署 LAMP 堆栈
description: 使用自定义脚本扩展在 Azure 上的负载均衡虚拟机规模集中部署 LAMP 堆栈。
services: virtual-machines-linux
documentationcenter: virtual-machines
tags: azure-service-management
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.custom: mvc, devx-track-azurecli
origin.date: 04/05/2017
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: b01f7e758e8c1bb2bf288363ee449d67cc97a75c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104535"
---
# <a name="deploy-the-lamp-stack-in-a-load-balanced-virtual-machine-scale-set"></a>在负载均衡虚拟机规模集中部署 LAMP 堆栈

此示例创建一个虚拟机规模集，并应用运行自定义脚本的扩展在规模集中每个虚拟机上部署 LAMP 堆栈。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/bash

# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login 

# Create the resource group if it doesn't exist
az group create -n myResourceGroup -l chinanorth

# Build the Scale Set
az vmss create -n myScaleSet -g myResourceGroup --public-ip-address-dns-name my-lamp-sample \
    --image CentOS --storage-sku Premium_LRS --admin-username deploy --vm-sku Standard_DS3_v2

# Add a load balanced endpoint on port 80 routed to the backend servers on port 80
az network lb rule create -g myResourceGroup -n http-rule --backend-pool-name myScaleSetLBBEPool \
    --backend-port 80 --frontend-ip-name LoadBalancerFrontEnd --frontend-port 80 --lb-name myScaleSetLB \
    --protocol Tcp

# Create a virtual machine scale set custom script extension. This extension will provide configuration
# to each of the virtual machines within the scale set on how to provision their software stack.
# The configuration (./projected_config.json) contains commands to be executed upon provisioning
# of instances. This is helpful for hooking into configuration management software or simply
# provisioning your software stack directly.
az vmss extension set -n CustomScript --publisher Microsoft.Azure.Extensions --version 2.0 \
   -g myResourceGroup --vmss-name myScaleSet --protected-settings ./protected_config.json --no-auto-upgrade

# The instances that we have were created before the extension was added.
# Update these instances to run the new configuration on each of them.
az vmss update-instances --instance-ids "*" -n myScaleSet -g myResourceGroup

# Scaling adds new instances. These instances will run the configuration when they're provisioned.
az vmss scale --new-capacity 2 -n myScaleSet -g myResourceGroup

```

## <a name="connect"></a>连接

使用此代码查看如何连接到 VM 和规模集。

```azurecli
#!/bin/bash

FQDN=$(az network public-ip list -g myResourceGroup --query \
                "[?starts_with(dnsSettings.fqdn, 'my-lamp-')].dnsSettings.fqdn | [0]" -o tsv)

PORTS=$(az network lb show -g myResourceGroup -n myScaleSetLB \
    --query "inboundNatRules[].{backend: backendPort, frontendPort: frontendPort}" -o tsv)
while read CMD; do
    read _ frontend <<< "${CMD}"
    echo "'ssh deploy@${FQDN} -p ${frontend}'"
done <<< "${PORTS}"

echo ""
echo "You can now reach the scale set by opening your browser to: 'http://${FQDN}'."
```

## <a name="clean-up-deployment"></a>清理部署 

运行如下命令来删除资源组、规模集和 VM 以及所有相关资源。

```azurecli 
az group delete -n myResourceGroup
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机、可用性集、负载均衡器和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az vmss create](https://docs.azure.cn/cli/vmss?view=azure-cli-latest#az-vmss-create) | 创建虚拟机规模集 |
| [az network lb rule create](https://docs.azure.cn/cli/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) | 添加负载均衡终结点 |
| [az vmss extension set](https://docs.azure.cn/cli/vmss/extension?view=azure-cli-latest#az-vmss-extension-set) | 创建对 VM 部署运行自定义脚本的扩展 |
| [az vmss update-instances](https://docs.azure.cn/cli/vmss?view=azure-cli-latest#az-vmss-update-instances) | 在将扩展应用到规模集之前部署的 VM 实例上运行此自定义脚本。 |
| [az vmss scale](https://docs.azure.cn/cli/vmss?view=azure-cli-latest#az-vmss-scale) | 通过添加更多 VM 实例来扩大规模集。 部署实例时，在实例上运行此自定义脚本。 |
| [az network public-ip list](https://docs.azure.cn/cli/network/public-ip?view=azure-cli-latest#az-network-public-ip-list) | 获取示例创建的 VM 的 IP 地址。 |
| [az network lb show](https://docs.azure.cn/cli/network/lb?view=azure-cli-latest#az-network-lb-show) | 获取负载均衡器使用的前端和后端端口。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.azure.cn/cli/index?view=azure-cli-latest)。

可以在 [Azure Linux VM 文档](../linux/cli-samples.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)中找到其他虚拟机 CLI 脚本示例。

<!--Update_Description: update meta properties -->