---
title: 使用门户或 PowerShell 创建虚拟硬盘驱动器的快照
description: 了解如何使用门户或 PowerShell 创建 Azure VM 的副本来将其用作备份或用于排查问题。
manager: twooley
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
origin.date: 10/08/2018
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.subservice: disks
ms.openlocfilehash: b4b04c614158f71d18b5648c954f87154abe222c
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128362"
---
# <a name="create-a-snapshot-using-the-portal-or-powershell"></a>使用门户或 PowerShell 创建快照

快照是虚拟硬盘 (VHD) 的完整只读副本。 可以创建 OS 或数据磁盘 VHD 的快照以用作备份，或用于解决虚拟机 (VM) 问题。

如果要使用快照创建新的 VM，建议在拍摄快照前彻底关闭 VM，以清除正在运行的所有进程。

## <a name="use-the-azure-portal"></a>使用 Azure 门户 

若要创建快照，请完成以下步骤： 
1. 在 [Azure 门户](https://portal.azure.cn)中，选择“创建资源”。
2. 搜索并选择“快照”。
3. 在“快照”窗口，选择“创建” 。 此时将显示“创建快照”窗口。
4. 输入快照的 **名称** 。
5. 选择现有的[资源组](../../azure-resource-manager/management/overview.md#resource-groups)，或键入新资源组的名称。 
6. 选择 Azure 数据中心“位置” 。  
7. 对于 **源磁盘** ，选择要获取其快照的托管磁盘。
8. 选择用于存储快照的“帐户类型”。 选择“Standard_HDD”，除非需要将快照存储在高性能磁盘上。
9. 选择“创建” 。

## <a name="use-powershell"></a>使用 PowerShell

以下步骤显示如何复制 VHD 磁盘并创建快照配置。 然后，可以使用 [New-AzSnapshot](https://docs.microsoft.com/powershell/module/az.compute/new-azsnapshot) cmdlet 创建磁盘快照。 

1. 设置一些参数： 

    ```powershell
    $resourceGroupName = 'myResourceGroup' 
    $location = 'chinaeast' 
    $vmName = 'myVM'
    $snapshotName = 'mySnapshot'  
    ```

2. 获取 VM：

    ```powershell
    $vm = Get-AzVM `
        -ResourceGroupName $resourceGroupName `
        -Name $vmName
    ```

3. 创建快照配置。 该示例中，此快照是 OS 磁盘的快照：

    ```powershell
    $snapshot =  New-AzSnapshotConfig `
        -SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
        -Location $location `
        -CreateOption copy
    ```

    <!--Not Available on [availability zones](../../availability-zones/az-overview.md)-->
    <!--Not Available on FEATURE Standard_ZRS-->

4. 拍摄快照：

    ```powershell
    New-AzSnapshot `
        -Snapshot $snapshot `
        -SnapshotName $snapshotName `
        -ResourceGroupName $resourceGroupName 
    ```

## <a name="next-steps"></a>后续步骤

通过从快照创建托管磁盘，然后将新的托管磁盘附加为 OS 磁盘，来从快照创建虚拟机。 有关详细信息，请参阅[使用 PowerShell 从快照创建 VM](./../scripts/virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md) 中的示例。

<!-- Update_Description: update meta properties, wording update, update link -->