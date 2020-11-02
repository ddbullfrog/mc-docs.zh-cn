---
title: 教程 - 使用 Azure PowerShell 创建和管理 Windows VM
description: 本教程介绍如何使用 Azure PowerShell 在 Azure 中创建和管理 Windows VM
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
origin.date: 06/06/2019
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: a8e31e647fc9a8736e660e19a77b5f44573fef51
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103562"
---
# <a name="tutorial-create-and-manage-windows-vms-with-azure-powershell"></a>教程：使用 Azure PowerShell 创建和管理 Windows VM

Azure 虚拟机提供完全可配置的灵活计算环境。 本教程介绍 Azure 虚拟机 (VM) 的基本部署任务，例如选择 VM 大小、选择 VM 映像和部署 VM。 你将学习如何执行以下操作：

> [!div class="checklist"]
> * 创建并连接到 VM
> * 选择并使用 VM 映像
> * 查看和使用特定 VM 大小
> * 调整 VM 的大小
> * 查看并了解 VM 状态

## <a name="launch-azure-local-powershell"></a>启动 Azure 本地 PowerShell

打开 Azure Powershell 控制台，并以管理员权限运行以下脚本。

<!--Not Available on Azure Cloud Shell-->

## <a name="create-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 命令创建资源组。

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 必须在创建虚拟机前创建资源组。 在以下示例中，在“ChinaEast”区域中创建了名为“myResourceGroupVM”的资源组：

```powershell
# Sign in the Azure China Cloud
Connect-AzAccount -Environment AzureChinaCloud

New-AzResourceGroup `
   -ResourceGroupName "myResourceGroupVM" `
   -Location "ChinaEast"
```

创建或修改 VM 时指定资源组，本教程会对此进行演示。

## <a name="create-a-vm"></a>创建 VM

创建 VM 时，可使用多个选项，例如操作系统映像、网络配置和管理凭据。 此示例创建名为 *myVM* 的 VM，运行默认版本的 Windows Server 2016 Datacenter。

使用 [Get-Credential](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-credential?view=powershell-6) 设置 VM 上管理员帐户所需的用户名和密码：

```powershell
$cred = Get-Credential
```

使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Location "ChinaEast" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred
```

## <a name="connect-to-vm"></a>连接到 VM

在部署完成后，创建到 VM 的远程桌面连接。

运行以下命令，以返回 VM 的公共 IP 地址。 需记下此 IP 地址，以便在后续步骤中使用浏览器连接到它测试 Web 连接。

```powershell
Get-AzPublicIpAddress `
   -ResourceGroupName "myResourceGroupVM"  | Select IpAddress
```

在本地计算机上使用以下命令创建与 VM 的远程桌面会话。 将 IP 地址替换为你的 VM 的 *publicIPAddress* 。 出现提示时，输入创建 VM 时使用的凭据。

```powershell
mstsc /v:<publicIpAddress>
```

在“Windows 安全性”  窗口中，依次选择“更多选择”  、“使用其他帐户”  。 键入针对 VM 创建的用户名和密码，然后单击“确定”。

## <a name="understand-marketplace-images"></a>了解市场映像

Azure 市场包括许多可用于新建 VM 的映像。 在之前的步骤中，使用 Windows Server 2016 Datacenter 映像创建了 VM。 在此步骤中，我们将使用 PowerShell 模块在市场中搜索其他 Windows 映像，这些映像也可用作新 VM 的基础。 此过程包括查找发布者、产品/服务、SKU，以及用于[标识](cli-ps-findimage.md#terminology)映像的版本号（可选）。

使用 [Get-AzVMImagePublisher](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimagepublisher) 命令返回映像发布者的列表：

```powershell
Get-AzVMImagePublisher -Location "ChinaEast"
```

使用 [Get-AzVMImageOffer](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimageoffer) 返回映像产品的列表。 使用此命令，返回筛选了指定发布者（名为 `MicrosoftWindowsServer`）的列表：

```powershell
Get-AzVMImageOffer `
   -Location "ChinaEast" `
   -PublisherName "MicrosoftWindowsServer"
```

结果将如以下示例所示： 

```powershell
Offer             PublisherName          Location
-----             -------------          --------
Windows-HUB       MicrosoftWindowsServer ChinaEast
WindowsServer     MicrosoftWindowsServer ChinaEast
WindowsServer-HUB MicrosoftWindowsServer ChinaEast
```

然后，使用 [Get-AzVMImageSku](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimagesku) 命令对发布者和产品名称进行筛选，以返回映像名称的列表。

```powershell
Get-AzVMImageSku `
   -Location "ChinaEast" `
   -PublisherName "MicrosoftWindowsServer" `
   -Offer "WindowsServer"
```

结果将如以下示例所示： 

```powershell
Skus                                      Offer         PublisherName          Location
----                                      -----         -------------          --------
2008-R2-SP1                               WindowsServer MicrosoftWindowsServer ChinaEast  
2008-R2-SP1-smalldisk                     WindowsServer MicrosoftWindowsServer ChinaEast  
2012-Datacenter                           WindowsServer MicrosoftWindowsServer ChinaEast  
2012-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer ChinaEast  
2012-R2-Datacenter                        WindowsServer MicrosoftWindowsServer ChinaEast  
2012-R2-Datacenter-smalldisk              WindowsServer MicrosoftWindowsServer ChinaEast  
2016-Datacenter                           WindowsServer MicrosoftWindowsServer ChinaEast  
2016-Datacenter-Server-Core               WindowsServer MicrosoftWindowsServer ChinaEast  
2016-Datacenter-Server-Core-smalldisk     WindowsServer MicrosoftWindowsServer ChinaEast
2016-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer ChinaEast
2016-Datacenter-with-Containers           WindowsServer MicrosoftWindowsServer ChinaEast
2016-Datacenter-with-Containers-smalldisk WindowsServer MicrosoftWindowsServer ChinaEast
2016-Datacenter-with-RDSH                 WindowsServer MicrosoftWindowsServer ChinaEast
2016-Nano-Server                          WindowsServer MicrosoftWindowsServer ChinaEast
```

此信息可用于部署具有特定映像的 VM。 此示例通过将最新版本的 Windows Server 2016 与容器映像配合使用来部署 VM。

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM2" `
    -Location "ChinaEast" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress2" `
    -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
    -Credential $cred `
    -AsJob
```

`-AsJob` 参数以后台任务的方式创建 VM，因此 PowerShell 提示符会返回到你所在的位置。 可以通过 `Get-Job` cmdlet 查看后台作业的详细信息。

## <a name="understand-vm-sizes"></a>了解 VM 大小

VM 大小决定 VM 可用计算资源（如 CPU、GPU 和内存）的数量。 创建的虚拟机大小应适合工作负荷。 如果工作负荷增加，也可重设现有虚拟机的大小。

### <a name="vm-sizes"></a>VM 大小

下表将大小分类成了多个用例。  

| 类型                     | 常见大小           |    说明       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [常规用途](../sizes-general.md)         |B、Dsv3、Dv3、DSv2、Dv2、Av2 | CPU 与内存之比均衡。 适用于开发/测试、小到中型应用程序和数据解决方案。  |
| [计算优化](../sizes-compute.md)   | Fsv2          | 高 CPU 与内存之比。 适用于中等流量的应用程序、网络设备和批处理。        |
| [内存优化](../sizes-memory.md)    | Esv3、Ev3、M、DSv2、Dv2  | 较高的内存核心比。 适用于关系数据库、中到大型缓存和内存中分析。                 |
| [GPU](../sizes-gpu.md)          |  NCv3        | 专门针对大量图形绘制和视频编辑的 VM。       |

<!-- Not Available on DC Series -->
<!-- Not Available on Lsv2, Ls   -->
<!-- Not Available on [GPU](sizes-gpu.md)-->
<!-- Not Available on [High performance](sizes-hpc.md)-->


### <a name="find-available-vm-sizes"></a>查找可用的 VM 大小

若要查看在特定区域可用的 VM 大小的列表，请使用 [Get-AzVMSize](https://docs.microsoft.com/powershell/module/az.compute/get-azvmsize) 命令。

```powershell
Get-AzVMSize -Location "ChinaEast"
```

## <a name="resize-a-vm"></a>调整 VM 的大小

部署 VM 后，可调整其大小以增加或减少资源分配。

调整 VM 大小之前，请检查所需的大小在当前 VM 群集上是否可用。 使用 [Get AzVMSize](https://docs.microsoft.com/powershell/module/az.compute/get-azvmsize) 命令返回大小的列表。

```powershell
Get-AzVMSize -ResourceGroupName "myResourceGroupVM" -VMName "myVM"
```

如果大小可用，则可从开机状态调整 VM 大小，但需在此操作期间重启 VM。

```powershell
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM `
   -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
```

如果所需大小在当前群集上不可用，则需解除分配 VM，才能执行重设大小操作。 解除分配 VM 时会删除临时磁盘上的任何数据，并且如果不使用静态 IP 地址，则公共 IP 地址会发生更改。

```powershell
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_E2s_v3"
Update-AzVM -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -Name $vm.name
```

## <a name="vm-power-states"></a>VM 电源状态

Azure VM 可能会处于多种电源状态之一。 

| 电源状态 | 说明
|----|----|
| 正在启动 | 正在启动虚拟机。 |
| 正在运行 | 虚拟机正在运行。 |
| 正在停止 | 正在停止虚拟机。 |
| 已停止 | VM 已停止。 虚拟机处于停止状态时仍会产生计算费用。  |
| 正在解除分配 | VM 正解除分配。 |
| 已解除分配 | 指示 VM 已从监控程序中删除，但仍可在控制面板中使用。 处于 `Deallocated` 状态的虚拟机不会产生计算费用。 |
| - | VM 的电源状态未知。 |

若要获取特定 VM 的状态，请使用 [Get-AzVM](https://docs.microsoft.com/powershell/module/az.compute/get-azvm) 命令。 请确保为 VM 和资源组指定有效的名称。

```powershell
Get-AzVM `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Status | Select @{n="Status"; e={$_.Statuses[1].Code}}
```

输出将如以下示例所示：

```powershell
Status
------
PowerState/running
```

若要检索订阅中所有 VM 的电源状态，请使用[虚拟机 - 列出所有 API](https://docs.microsoft.com/rest/api/compute/virtualmachines/listall)，并将参数 statusOnly 设置为 true。

## <a name="management-tasks"></a>管理任务

在 VM 生命周期中，可能需要运行管理任务，例如启动、停止或删除 VM。 此外，可能还需要创建脚本来自动执行重复或复杂的任务。 使用 Azure PowerShell，可从命令行或脚本运行许多常见的管理任务。

### <a name="stop-a-vm"></a>停止 VM

使用 [Stop-AzVM](https://docs.microsoft.com/powershell/module/az.compute/stop-azvm) 停止并解除分配 VM：

```powershell
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
```

若要让 VM 保持已预配状态，请使用 -StayProvisioned 参数。

### <a name="start-a-vm"></a>启动 VM

```powershell
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM"
```

### <a name="delete-resource-group"></a>删除资源组

删除资源组时，会删除其中的一切。

```powershell
Remove-AzResourceGroup `
   -Name "myResourceGroupVM" `
   -Force
```

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习 VM 创建和管理的基本知识，例如如何：

> [!div class="checklist"]
> * 创建并连接到 VM
> * 选择并使用 VM 映像
> * 查看和使用特定 VM 大小
> * 调整 VM 的大小
> * 查看并了解 VM 状态

请转到下一教程，了解 VM 磁盘。  

> [!div class="nextstepaction"]
> [创建和管理 VM 磁盘](./tutorial-manage-data-disk.md)

<!-- Update_Description: update meta properties, wording update, update link -->