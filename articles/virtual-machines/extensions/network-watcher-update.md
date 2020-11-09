---
title: 将网络观察程序扩展更新到最新版本
description: 了解如何将 Azure 网络观察程序扩展更新到最新版本。
services: virtual-machines-windows
manager: balar
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.topic: article
ms.workload: infrastructure-services
origin.date: 09/23/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: ffb104283c264776b666ee810c89a3af3fab7706
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106800"
---
<!--Verified Successfully-->
# <a name="update-the-network-watcher-extension-to-the-latest-version"></a>将网络观察程序扩展更新到最新版本

## <a name="overview"></a>概述

[Azure 网络观察程序](../../network-watcher/network-watcher-monitoring-overview.md)是一项网络性能监视、诊断和分析服务，可对 Azure 网络进行监视。 若要按需捕获网络流量和使用 Azure VM 上的其他高级功能，必须具备网络观察程序代理虚拟机 (VM) 扩展。 网络观察程序扩展用于连接监视器、连接监视器（预览版）、连接疑难解答和数据包捕获等功能。

## <a name="prerequisites"></a>先决条件

本文假设你已在 VM 中安装了网络观察程序扩展。

## <a name="latest-version"></a>最新版本

当前，网络观察程序扩展的最新版本为 `1.4.1654.1`。

## <a name="update-your-extension"></a>更新扩展

若要更新扩展，需要知道扩展版本。

### <a name="check-your-extension-version"></a>检查扩展版本

可使用 Azure 门户、Azure CLI 或 PowerShell 检查扩展版本。

#### <a name="usetheazureportal"></a>使用 Azure 门户

1. 在 Azure 门户中转到 VM 的“扩展”窗格。
1. 选择“AzureNetworkWatcher”扩展，查看“详细信息”窗格。
1. 在“版本”字段中找到版本号。  

#### <a name="use-the-azure-cli"></a>使用 Azure CLI

在 Azure CLI 提示符下运行以下命令：

```azurecli
az vm extension list --resource-group  <ResourceGroupName> --vm-name <VMName>
```

在输出中找到 AzureNetworkWatcher 扩展。 在输出的“TypeHandlerVersion”字段中识别版本号。  

#### <a name="usepowershell"></a>使用 PowerShell

在 PowerShell 提示符下运行以下命令：

```powershell
Get-AzVMExtension -ResourceGroupName <ResourceGroupName> -VMName <VMName>  
```

在输出中找到 AzureNetworkWatcher 扩展。 在输出的“TypeHandlerVersion”字段中识别版本号。

### <a name="update-your-extension"></a>更新扩展

如果你的版本比当前的最新版本 `1.4.1654.1` 低，请使用以下任一选项更新扩展。

#### <a name="option-1-use-powershell"></a>选项 1：使用 PowerShell

运行以下命令：

```powershell
#Linux command
Set-AzVMExtension `  -ResourceGroupName "myResourceGroup1" `  -Location "ChinaNorth" `  -VMName "myVM1" `  -Name "AzureNetworkWatcherExtension" `  -Publisher "Microsoft.Azure.NetworkWatcher" -Type "NetworkWatcherAgentLinux"   

#Windows command
Set-AzVMExtension `  -ResourceGroupName "myResourceGroup1" `  -Location "ChinaNorth" `  -VMName "myVM1" `  -Name "AzureNetworkWatcherExtension" `  -Publisher "Microsoft.Azure.NetworkWatcher" -Type "NetworkWatcherAgentWindows"   
```

#### <a name="option-2-use-the-azure-cli"></a>选项 2：使用 Azure CLI

强制执行升级。

```azurecli
#Linux command
az vm extension set --resource-group "myResourceGroup1" --vm-name "myVM1" --name "NetworkWatcherAgentLinux" --publisher "Microsoft.Azure.NetworkWatcher" --force-update

#Windows command
az vm extension set --resource-group "myResourceGroup1" --vm-name "myVM1" --name "NetworkWatcherAgentWindows" --publisher "Microsoft.Azure.NetworkWatcher" --force-update
```

<!--MOONCAKE FAILED AT CLI command: #Linux command-->

如果不起作用，请删除再重新安装该扩展，然后按照以下步骤自动添加最新版本。

删除扩展.

```azurecli
#Same for Linux and Windows
az vm extension delete --resource-group "myResourceGroup1" --vm-name "myVM1" -n "AzureNetworkWatcherExtension"

```

再次安装该扩展。

```azurecli
#Linux command
az vm extension set --resource-group "DALANDEMO" --vm-name "Linux-01" --name "NetworkWatcherAgentLinux" --publisher "Microsoft.Azure.NetworkWatcher"  

#Windows command
az vm extension set --resource-group "DALANDEMO" --vm-name "Linux-01" --name "NetworkWatcherAgentWindows" --publisher "Microsoft.Azure.NetworkWatcher" 

```

#### <a name="option-3-reboot-your-vms"></a>选项 3：重启 VM

如果已将网络观察程序扩展的自动升级设置为 True，请重启 VM 安装以获取最新扩展。

## <a name="support"></a>支持

如果在本文的任何位置需要更多帮助，请参阅 [Linux](./network-watcher-linux.md) 或 [Windows](./network-watcher-windows.md) 的网络观察程序扩展文档。 还可通过 [Azure 支持](https://support.azure.cn/support/contact/)联系 Azure 专家。 或者，提交 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/support/support-azure/)提交请求。 有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](https://www.azure.cn/support/faq/)。

<!-- Update_Description: new article about network watcher update -->
<!--NEW.date: 11/02/2020-->