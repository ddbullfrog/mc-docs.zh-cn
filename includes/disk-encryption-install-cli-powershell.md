---
title: include 文件
description: include 文件
services: virtual-machines
ms.service: virtual-machines
ms.topic: include
origin.date: 10/06/2019
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 6b4c95b54a7ffa4aaf930938ffed7ab3283ce50d
ms.sourcegitcommit: 221c32fe6f618679a63f148da7382bc9e495f747
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211879"
---
可以通过 [Azure CLI](https://docs.azure.cn/cli/) 和 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/new-azureps-module-az) 启用和管理 Azure 磁盘加密。 为此，必须在本地安装工具并连接到 Azure 订阅。

### <a name="azure-cli"></a>Azure CLI

[Azure CLI 2.0](https://docs.azure.cn/cli/) 是用于管理 Azure 资源的命令行工具。 CLI 旨在提高数据查询灵活性、支持非阻塞进程形式的长时间操作，以及简化脚本编写。 可以按照[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli) 中的步骤在本地安装它。

若要[使用 Azure CLI 登录 Azure 帐户](https://docs.azure.cn/cli/authenticate-azure-cli)，请使用 [az login](https://docs.azure.cn/cli/reference-index#az_login) 命令。

```azurecli
az cloud set -n AzureChinaCloud
az login
```

若要选择登录到的租户，请使用：

```azurecli
az login --tenant <tenant>
```

如果有多个订阅并想要指定其中的一个，请使用 [az account list](https://docs.azure.cn/cli/account#az_account_list) 获取订阅列表，然后使用 [az account set](https://docs.azure.cn/cli/account#az_account_set) 指定订阅。

```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

有关详细信息，请参阅 [Azure CLI 2.0 入门](https://docs.azure.cn/cli/get-started-with-azure-cli)。 

### <a name="azure-powershell"></a>Azure PowerShell
[Azure PowerShell az 模块](https://docs.microsoft.com/powershell/azure/new-azureps-module-az)提供了一组使用 [Azure 资源管理器](https://docs.azure.cn/azure-resource-manager/resource-group-overview)模型管理 Azure 资源的 cmdlet。 可以按照[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-az-ps)中的说明在本地计算机上安装它。 

<!--Not Available on [Azure Cloud Shell](https://docs.microsoft.com/cloud-shell/overview)-->

如果已在本地安装 PowerShell，请确保使用最新版本的 Azure PowerShell SDK 来配置 Azure 磁盘加密。 下载最新版本的 [Azure PowerShell 版本](https://github.com/Azure/azure-powershell/releases)。

若要[使用 Azure PowerShell 登录 Azure 帐户](https://docs.microsoft.com/powershell/azure/authenticate-azureps?view=azps-2.5.0)，请使用 [Connect-AzAccount -Environment AzureChinaCloud](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) cmdlet。

```powershell
Connect-AzAccount -Environment AzureChinaCloud
```

如果有多个订阅并想要指定一个，请使用 [Get-AzSubscription](https://docs.microsoft.com/powershell/module/Az.Accounts/Get-AzSubscription) cmdlet 列出这些订阅，然后使用 [Set-AzContext](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext?view=azps-2.5.0) cmdlet：

```powershell
Set-AzContext -Subscription -Subscription <SubscriptionId>
```

运行 [Get-AzContext](https://docs.microsoft.com/powershell/module/Az.Accounts/Get-AzContext) cmdlet 将验证是否选择了正确的订阅。

若要确认已安装 Azure 磁盘加密 cmdlet，请使用 [Get-command](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/get-command?view=powershell-6) cmdlet：

```powershell
Get-command *diskencryption*
```
有关详细信息，请参阅 [Azure PowerShell 入门](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

<!-- Update_Description: update meta properties, wording update, update link -->