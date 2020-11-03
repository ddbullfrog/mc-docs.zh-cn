---
title: 使用 PowerShell 将数据磁盘附加到 Azure 中的 Windows VM
description: 如何配合使用 PowerShell 和 Resource Manager 部署模型将新磁盘或现有数据磁盘附加到 Windows VM。
ms.service: virtual-machines-windows
ms.topic: how-to
origin.date: 10/16/2018
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.subservice: disks
ms.openlocfilehash: af7590ba962d1105a3cf2df3730f3e3cb390319a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106126"
---
# <a name="attach-a-data-disk-to-a-windows-vm-with-powershell"></a>使用 PowerShell 将数据磁盘附加到 Windows VM

本文介绍了如何使用 PowerShell 将新磁盘和现有磁盘附加到 Windows 虚拟机。 

首先，查看以下提示：

* 虚拟机的大小决定了可以附加多少个磁盘。 有关详细信息，请参阅[虚拟机的大小](../sizes.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)。
* 若要使用高级 SSD，需要[支持高级存储的 VM 类型](../sizes-memory.md)，如 DS 系列虚拟机。

<!-- Not Available on GS-series -->

<!--Not Available on [Azure Cloud Shell](../../cloud-shell/overview.md)-->

## <a name="add-an-empty-data-disk-to-a-virtual-machine"></a>将空数据磁盘添加到虚拟机

此示例演示了如何将空数据磁盘添加到现有虚拟机。

### <a name="using-managed-disks"></a>使用托管磁盘

```powershell
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'China East' 
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

<!-- Not Available on ### Using managed disks in an Availability Zone-->

### <a name="initialize-the-disk"></a>初始化磁盘

添加空磁盘后，需要对其进行初始化。 要初始化该磁盘，可以登录到一个 VM，并使用磁盘管理进行初始化。 如果在创建 VM 时在其上启用了 [WinRM](https://docs.microsoft.com/windows/desktop/WinRM/portal) 和证书，则可以使用远程 PowerShell 初始化该磁盘。 还可以使用自定义脚本扩展：

```powershell
$location = "location-name"
$scriptName = "script-name"
$fileName = "script-file-name"
Set-AzVMCustomScriptExtension -ResourceGroupName $rgName -Location $locName -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"
```

脚本文件可以包含用来初始化磁盘的代码，例如：

```powershell
$disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

$letters = 70..89 | ForEach-Object { [char]$_ }
$count = 0
$labels = "data1","data2"

foreach ($disk in $disks) {
    $driveLetter = $letters[$count].ToString()
    $disk | 
    Initialize-Disk -PartitionStyle MBR -PassThru |
    New-Partition -UseMaximumSize -DriveLetter $driveLetter |
    Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
    $count++
}
```

## <a name="attach-an-existing-data-disk-to-a-vm"></a>将现有数据磁盘附加到 VM

可以将现有的托管磁盘作为数据磁盘附加到虚拟机。

```powershell
$rgName = "myResourceGroup"
$vmName = "myVM"
$location = "China East" 
$dataDiskName = "myDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## <a name="next-steps"></a>后续步骤

还可以使用模板部署托管磁盘。 有关详细信息，请参阅[使用 Azure 资源管理器模板中的托管磁盘](../using-managed-disks-template-deployments.md)或[快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-multiple-data-disk)以部署多个数据磁盘。

<!-- Update_Description: update meta properties, wording update, update link -->