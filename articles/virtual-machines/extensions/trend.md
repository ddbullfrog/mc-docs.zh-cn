---
title: 在 VM 上安装 Trend Micro Deep Security
description: 本文介绍如何在 Azure 中使用经典部署模型创建的 VM 上安装和配置 Trend Micro Deep Security。
tags: azure-service-management
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: article
origin.date: 04/20/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 5dc2c3aeda51a9a330c33103772b20a6328f0f76
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104877"
---
# <a name="how-to-install-and-configure-trend-micro-deep-security-as-a-service-on-a-windows-vm"></a>如何在 Windows VM 上安装和配置 Trend Micro Deep Security 即服务

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

[!INCLUDE [virtual-machines-extensions-deprecation-statement](../../../includes/virtual-machines-extensions-deprecation-statement.md)]
本文演示了如何在运行 Windows Server 的新的或现有虚拟机 (VM) 上安装和配置 Trend Micro Deep Security 即服务。 Deep Security 即服务包括反恶意软件保护、防火墙、入侵防御系统和完整性监视。

该客户端通过 VM 代理作为安全扩展插件进行安装。 在新虚拟机上安装 Deep Security Agent，因为 VM 代理由 Azure 门户自动创建。

使用 Azure 门户、Azure CLI 或 PowerShell 创建的现有 VM 可能不具有 VM 代理。 对于未安装 VM 代理的现有虚拟机，需要先下载并安装 VM 代理。 本文介绍这两种情况。

如果已经有针对本地解决方案的 Trend Micro 当前订阅，则可用它来帮助保护 Azure 虚拟机。 如果还不是客户，则可注册试用订阅。 有关此解决方案的详细信息，请参阅 Trend Micro 博客文章 [Azure VM Agent Extension For Deep Security](https://go.microsoft.com/fwlink/p/?LinkId=403945)（用于 Deep Security 的 Azure VM 代理扩展）。


<!--Not Available on ## Install the Deep Security Agent on a new VM-->


## <a name="install-the-deep-security-agent-on-an-existing-vm"></a>在现有 VM 上安装 Deep Security Agent
若要在现有 VM 上安装代理，需要以下各项：

* 在本地计算机上安装 Azure PowerShell 模块 0.8.2 版或更高版本。 可以使用 **Get-Module azure | format-table version** 命令查看已安装的 Azure PowerShell 的版本。 有关说明以及指向最新版本的链接，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azure/)。 使用 `Add-AzureAccount -Environment AzureChinaCloud` 登录到 Azure 订阅。
* 在目标虚拟机上安装 VM 代理。

首先，请验证是否已安装 VM 代理。 填写云服务名称和虚拟机名称，并在管理员级别的 Azure PowerShell 命令提示符下运行以下命令。 替换引号内的所有内容，包括 < and > 字符。

```azurepowershell
$CSName = "<cloud service name>"
$VMName = "<virtual machine name>"
$vm = Get-AzureVM -ServiceName $CSName -Name $VMName
write-host $vm.VM.ProvisionGuestAgent
```

如果不知道云服务名称和虚拟机名称，运行 **Get-AzureVM** 即可显示当前订阅中所有虚拟机的该信息。

如果 **write-host** 命令返回 **True** ，则已安装 VM 代理。 如果该命令返回 **False** ，请参阅 Azure 博客文章 [VM 代理和扩展 - 第 2 部分](https://go.microsoft.com/fwlink/p/?LinkId=403947)中的说明和下载链接。

如果已安装 VM 代理，请运行以下命令。

```azurepowershell
$Agent = Get-AzureVMAvailableExtension TrendMicro.DeepSecurity -ExtensionName TrendMicroDSA

Set-AzureVMExtension -Publisher TrendMicro.DeepSecurity -Version $Agent.Version -ExtensionName TrendMicroDSA -VM $vm | Update-AzureVM
```

## <a name="next-steps"></a>后续步骤
安装后，代理需要几分钟时间才会开始运行。 之后需在虚拟机上激活 Deep Security，才能通过 Deep Security Manager 进行管理。 有关其他说明，请参阅以下文章：

* 有关此解决方案的 Trend 文章： [Instant-On Cloud Security for Azure](https://go.microsoft.com/fwlink/?LinkId=404101)
* 用于配置虚拟机的 [Windows PowerShell 脚本示例](https://go.microsoft.com/fwlink/?LinkId=404100)
* 示例的[说明](https://go.microsoft.com/fwlink/?LinkId=404099)

## <a name="additional-resources"></a>其他资源
[如何登录到运行 Windows Server 的虚拟机]

[Azure VM 扩展和功能]

<!-- Image references -->

[1]: ./media/trend/new_vm_Blade3.png
[2]: ./media/trend/find_SecurityAgent.png
[3]: ./media/trend/SecurityAgentDetails.png

<!-- Link references -->

[如何登录到运行 Windows Server 的虚拟机]:../windows/classic/connect-logon.md
[Azure VM 扩展和功能]: features-windows.md

<!-- Update_Description: update meta properties, wording update, update link -->