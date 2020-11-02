---
title: Azure CLI 示例 - 创建运行 Azure Monitor 的 Azure VM
description: Azure CLI 示例 - 创建运行 Windows Server 2016 Datacenter 和 Azure Monitor 的 Azure VM。
services: virtual-machines-Windows
documentationcenter: virtual-machines
manager: gwallace
tags: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-Windows
ms.workload: infrastructure
origin.date: 02/23/2017
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc,seodec18, devx-track-azurecli
ms.openlocfilehash: c859b22d5d974d6da7026069afdc4e4034a2a532
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104170"
---
# <a name="monitor-a-vm-with-azure-monitor-logs"></a>使用 Azure Monitor 日志监视 VM

此脚本创建一个 Azure 虚拟机，安装 Log Analytics 代理，并将系统注册到 Log Analytics 工作区。 运行脚本后，该虚拟机会显示在 Azure Monitoring 中。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

```azurecli
#!/bin/sh

# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login

# Update for your admin password
AdminPassword=ChangeYourAdminPassword1

# OMS Id and OMS key.
omsid=<Replace with your OMS Id>
omskey=<Replace with your OMS key>

# Create a resource group.
az group create --name myResourceGroup --location chinanorth

# Create a virtual machine. 
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password $AdminPassword 

# Install and configure the OMS agent.
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM --name MicrosoftMonitoringAgent \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --version 1.0 --protected-settings '{"workspaceKey": "'"$omskey"'"}' \
  --settings '{"workspaceId": "'"$omsid"'"}'
```

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

```azurecli 
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 注释 |
|---|---|
| [az group create](https://docs.azure.cn/cli/group#az-group-create) | 创建用于存储所有资源的资源组。 |
| [az vm create](https://docs.azure.cn/cli/vm#az-vm-create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [azure vm extension set](https://docs.azure.cn/cli/vm/extension) | 针对虚拟机运行 VM 扩展。 |
| [az group delete](https://docs.azure.cn/cli/vm/extension#az-group-delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.azure.cn/cli)。

可以在 [Azure Windows VM 文档](../windows/cli-samples.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他虚拟机 CLI 脚本示例。

<!-- Update_Description: update meta properties, wording update, update link -->