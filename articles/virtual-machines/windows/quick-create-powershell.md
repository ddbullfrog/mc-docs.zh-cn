---
title: 快速入门 - 使用 Azure PowerShell 创建 Windows VM
description: 本快速入门介绍了如何使用 Azure PowerShell 创建 Windows 虚拟机
ms.service: virtual-machines-windows
ms.topic: quickstart
ms.workload: infrastructure
origin.date: 07/02/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: ff6c4852f65b2a338d6650de0136c19bffe60b40
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106285"
---
# <a name="quickstart-create-a-windows-virtual-machine-in-azure-with-powershell"></a>快速入门：使用 PowerShell 在 Azure 中创建 Windows 虚拟机

Azure PowerShell 模块用于从 PowerShell 命令行或脚本创建和管理 Azure 资源。 本快速入门演示如何使用 Azure PowerShell 模块在 Azure 中部署运行 Windows Server 2016 的虚拟机 (VM)。 若要显示运行中的 VM，也可通过 RDP 登录到该 VM 并安装 IIS Web 服务器。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="launch-azure-powershell"></a><a name="launch-azure-powershell"></a>启动 Azure PowerShell

<!--[!INCLUDE [cloud-shell-powershell](../../../includes/cloud-shell-powershell.md)]-->

可打开 Powershell 控制台，以管理员权限运行以下脚本。

## <a name="create-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。

```powershell
New-AzResourceGroup -Name myResourceGroup -Location ChinaEast
```

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 请提供每个资源的名称，如果这些资源不存在，`New-AzVM` cmdlet 会创建它们。

出现提示时，请提供用作 VM 登录凭据的用户名和密码：

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "China East" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80,3389
```

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

在部署完成后，通过 RDP 登录到 VM。 若要查看运行中的 VM，请安装 IIS Web 服务器。

若要查看 VM 的公共 IP 地址，请使用 [Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress) cmdlet：

```powershell
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

使用以下命令从本地计算机创建远程桌面会话。 将 IP 地址替换为你的 VM 的公共 IP 地址。 

```powershell
mstsc /v:publicIpAddress
```

在“Windows 安全性”窗口中，依次选择“更多选择”、“使用其他帐户”。 以 **localhost**\\*username* 的形式键入用户名，输入为虚拟机创建的密码，然后单击“确定”。

你可能会在登录过程中收到证书警告。 单击“是”或“继续”以创建连接 

## <a name="install-web-server"></a>安装 Web 服务器

若要查看运行中的 VM，请安装 IIS Web 服务器。 在 VM 中打开 PowerShell 提示符并运行以下命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成后，关闭到 VM 的 RDP 连接。

## <a name="view-the-web-server-in-action"></a>查看运行中的 Web 服务器

如果 IIS 已安装，并且 VM 上的端口 80 已对 Internet 开放， 则可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页。 使用上一步中获取的 VM 的公共 IP 地址。 以下示例展示了默认 IIS 网站：

:::image type="content" source="./media/quick-create-powershell/default-iis-website.png" alt-text="IIS 默认站点":::

## <a name="clean-up-resources"></a>清理资源

不再需要时，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) cmdlet 删除资源组、VM 和所有相关资源：

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你部署了简单的虚拟机，打开了 Web 流量的网络端口，并安装了一个基本 Web 服务器。 若要深入了解 Azure 虚拟机，请继续学习 Windows VM 教程。

> [!div class="nextstepaction"]
> [Azure Windows 虚拟机教程](./tutorial-manage-vm.md)

<!-- Update_Description: update meta properties, wording update, update link -->