---
title: 使用 PowerShell 更换 Azure VM 的 OS 磁盘
description: 使用 PowerShell 更改 Azure 虚拟机使用的操作系统磁盘。
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
origin.date: 04/24/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: no
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 6295f329d3a9a60214c630cab927cb60b2bc52c3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105852"
---
# <a name="change-the-os-disk-used-by-an-azure-vm-using-powershell"></a>使用 PowerShell 更改 Azure VM 使用的 OS 磁盘

如果当前具备 VM，但希望将磁盘更换为备份磁盘或其他 OS 磁盘，可使用 Azure PowerShell 更换 OS 磁盘。 无需删除和重新创建 VM。 甚至可在另一资源组中使用托管磁盘，只要该磁盘尚未使用。

需要停止/取消分配 VM，然后才可将该托管磁盘的资源 ID 替换为其他托管磁盘的资源 ID。

请确保 VM 大小和存储类型与要附加的磁盘兼容。 例如，如果要使用的磁盘位于高级存储中，则 VM 需要能使用高级存储（如 DS 系列大小）。 这两个磁盘的大小也必须相同。
此外，不支持将未加密的 VM 与加密的 OS 磁盘混合使用。 如果 VM 不使用 Azure 磁盘加密，则要交换的 OS 磁盘不应使用 Azure 磁盘加密。

使用 [Get-AzDisk](https://docs.microsoft.com/powershell/module/az.compute/get-azdisk) 获取资源组中的磁盘列表

```powershell
Get-AzDisk -ResourceGroupName myResourceGroup | Format-Table -Property Name
```

如果知道要使用的磁盘的名称，请将其设置为该 VM 的 OS 磁盘。 此示例停止/取消分配名为 myVM 的 VM，并将名为 newDisk 的磁盘分配为新的 OS 磁盘 。 

```powershell 
# Get the VM 
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM 

# Make sure the VM is stopped\deallocated
Stop-AzVM -ResourceGroupName myResourceGroup -Name $vm.Name -Force

# Get the new disk that you want to swap in
$disk = Get-AzDisk -ResourceGroupName myResourceGroup -Name newDisk

# Set the VM configuration to point to the new disk  
Set-AzVMOSDisk -VM $vm -ManagedDiskId $disk.Id -Name $disk.Name 

# Update the VM with the new OS disk
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm 

# Start the VM
Start-AzVM -Name $vm.Name -ResourceGroupName myResourceGroup

```

**后续步骤**

要创建磁盘副本，请参阅[拍摄磁盘快照](snapshot-copy-managed-disk.md)。

<!-- Update_Description: wording update -->