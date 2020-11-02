---
title: 在 Azure 中创建托管映像
description: 在 Azure 中创建通用 VM 或 VHD 的托管映像。 映像可用于创建多个使用托管磁盘的 VM。
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
origin.date: 09/27/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: legacy
ms.openlocfilehash: 449e927b263124b74ffcdceb3901c53c21ec92a2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104843"
---
# <a name="create-a-managed-image-of-a-generalized-vm-in-azure"></a>在 Azure 中创建通用 VM 的托管映像

可通过在存储帐户中存储为托管/非托管磁盘的通用虚拟机 (VM) 创建托管的映像资源。 然后可以使用该映像创建多个 VM。 有关托管映像如何计费的信息，请参阅[托管磁盘定价](https://www.azure.cn/pricing/details/storage/managed-disks/)。 

一个托管映像最多支持 20 个同时部署。 如果尝试从同一托管映像同时创建超过 20 个 VM，则可能会由于单个 VHD 的存储性能限制而导致预配超时。 若要同时创建 20 个以上的 VM，请使用为每 20 个并发 VM 部署配置 1 个副本的[共享映像库](shared-image-galleries.md)映像。

## <a name="generalize-the-windows-vm-using-sysprep"></a>使用 Sysprep 通用化 Windows VM

Sysprep 将删除所有个人帐户和安全信息，并准备好要用作映像的计算机。 有关 Sysprep 的信息，请参阅 [Sysprep 概述](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview)。

确保 Sysprep 支持计算机上运行的服务器角色。 有关详细信息，请参阅 [Sysprep 对服务器角色的支持](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles)和[不支持的方案](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview#unsupported-scenarios)。 

> [!IMPORTANT]
> 在 VM 上运行 Sysprep 后，该 VM 将被视为已通用化而无法重启。 通用化 VM 的过程是不可逆的。 如果需要保持原始 VM 正常运行，请创建 [VM 的副本](create-vm-specialized.md#option-3-copy-an-existing-azure-vm)并将其副本通用化。 
>
>Sysprep 要求对驱动器进行完全解密。 如果在 VM 上启用了加密，请在运行 Sysprep 之前将其禁用。
>
> 如果计划在首次将虚拟硬盘 (VHD) 上传到 Azure 之前运行 Sysprep，请确保先[准备好 VM](prepare-for-upload-vhd-image.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)。  
> 
> 

若要将 Windows VM 通用化，请执行以下步骤：

1. 登录到 Windows VM。

2. 以管理员身份打开“命令提示符”窗口。 

3. 删除 panther 目录 (C:\Windows\Panther)。 然后将目录切换到 %windir%\system32\sysprep，然后运行 `sysprep.exe`。

4. 在“系统准备工具”对话框中，选择“进入系统全新体验(OOBE)”，并选中“通用”复选框。

5. 在“关机选项”中选择“关机”。

6. 选择“确定”。

    :::image type="content" source="./media/upload-generalized-managed/sysprepgeneral.png" alt-text="启动 Sysprep":::

6. Sysprep 在完成运行后会关闭 VM。 不要重新启动 VM。

    > [!TIP]
    > **可选** 使用 [DISM](https://docs.microsoft.com/windows-hardware/manufacture/desktop/dism-optimize-image-command-line-options) 优化映像并减少 VM 的首次启动时间。
    >
    > 若要优化映像，请通过在 Windows 资源管理器中双击 VHD 来装载它，然后使用 `/optimize-image` 参数运行 DISM。
    >
    > ```cmd
    > DISM /image:D:\ /optimize-image /boot
    > ```
    > 其中 D：是装载的 VHD 的路径。
    >
    > 运行 `DISM /optimize-image` 应该是你对 VHD 所做的最后一次修改。 如果在部署之前对 VHD 进行了任何更改，则必须再次运行 `DISM /optimize-image`。

## <a name="create-a-managed-image-in-the-portal"></a>在门户中创建托管映像 

1. 转到 [Azure 门户](https://portal.azure.cn)以管理 VM 映像。 搜索并选择“虚拟机”。

2. 从列表中选择 VM。

3. 在 VM 的“虚拟机”页面的上方菜单中，选择“捕获”。

    将显示“创建映像”页面。

4. 对于“名称”，可以使用预填充的名称或输入想要为映像使用的名称。

5. 对于“资源组”，选择“新建”并输入名称，或者从下拉列表中选择要使用的资源组。

6. 如果想要在创建映像后删除源 VM，选择“创建映像后自动删除此虚拟机”。

    <!--Not Available on [availability zone](../../availability-zones/az-overview.md)-->
    
8. 选择“创建”以创建映像。

创建映像后，在资源组的资源列表中，你会看到它作为“映像”资源而出现。

## <a name="create-an-image-of-a-vm-using-powershell"></a>使用 PowerShell 创建 VM 映像

直接从 VM 创建映像，可确保映像包含所有与该 VM 关联的磁盘，包括 OS 磁盘和任何数据磁盘。 本示例演示如何从使用托管磁盘的 VM 创建托管映像。

在开始之前，请确保有最新版本的 Azure PowerShell 模块。 若要查找版本，请在 PowerShell 中运行 `Get-Module -ListAvailable Az`。 如需升级，请参阅[使用 PowerShellGet 在 Windows 上安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)。 如果在本地运行 PowerShell，则运行 `Connect-AzAccount -Environment AzureChinaCloud` 以创建与 Azure 的连接。

<!--Not Available on [availability zones](../../availability-zones/az-overview.md)-->

若要创建 VM 映像，请遵循下列步骤：

1. 创建一些变量。

    ```powershell
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "ChinaEast"
    $imageName = "myImage"
    ```
2. 确保 VM 已解除分配。

    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force
    ```

3. 将虚拟机的状态设置为“通用化”。 

    ```powershell
    Set-AzVm -ResourceGroupName $rgName -Name $vmName -Generalized
    ```

4. 获取虚拟机。 

    ```powershell
    $vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName
    ```

5. 创建映像配置。

    ```powershell
    $image = New-AzImageConfig -Location $location -SourceVirtualMachineId $vm.Id 
    ```
6. 创建映像。

    ```powershell
    New-AzImage -Image $image -ImageName $imageName -ResourceGroupName $rgName
    ``` 

## <a name="create-an-image-from-a-managed-disk-using-powershell"></a>使用 PowerShell 从托管磁盘创建映像

如果仅想创建 OS 磁盘的映像，则将托管磁盘 ID 指定为 OS 磁盘：

1. 创建一些变量。 

    ```powershell
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "ChinaEast"
    $imageName = "myImage"
    ```

2. 获取 VM。

    ```powershell
    $vm = Get-AzVm -Name $vmName -ResourceGroupName $rgName
    ```

3. 获取托管磁盘的 ID。

    ```powershell
    $diskID = $vm.StorageProfile.OsDisk.ManagedDisk.Id
    ```

3. 创建映像配置。

    ```powershell
    $imageConfig = New-AzImageConfig -Location $location
    $imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsState Generalized -OsType Windows -ManagedDiskId $diskID
    ```

4. 创建映像。

    ```powershell
    New-AzImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ``` 

## <a name="create-an-image-from-a-snapshot-using-powershell"></a>使用 PowerShell 从快照创建映像

通过执行以下步骤，可以从通用 VM 的快照创建托管映像：

1. 创建一些变量。 

    ```powershell
    $rgName = "myResourceGroup"
    $location = "ChinaEast"
    $snapshotName = "mySnapshot"
    $imageName = "myImage"
    ```

2. 获取快照。

    ```powershell
    $snapshot = Get-AzSnapshot -ResourceGroupName $rgName -SnapshotName $snapshotName
    ```

3. 创建映像配置。

    ```powershell
    $imageConfig = New-AzImageConfig -Location $location
    $imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsState Generalized -OsType Windows -SnapshotId $snapshot.Id
    ```
4. 创建映像。

    ```powershell
    New-AzImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ``` 

## <a name="create-an-image-from-a-vm-that-uses-a-storage-account"></a>从使用存储帐户的 VM 创建映像

若要从不使用托管磁盘的 VM 创建托管映像，需要存储帐户中 OS VHD 的 URI，格式如下： https:// *mystorageaccount* .blob.core.chinacloudapi.cn/ *vhdcontainer*/*vhdfilename.vhd* 。 在本示例中，VHD 位于名为 vhdcontainer 的容器中的 mystorageaccount 中，且 VHD 文件名为 vhdfilename.vhd。

1. 创建一些变量。

    ```powershell
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "ChinaEast"
    $imageName = "myImage"
    $osVhdUri = "https://mystorageaccount.blob.core.chinacloudapi.cn/vhdcontainer/vhdfilename.vhd"
    ```
2. 停止/解除分配 VM。

    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force
    ```

3. 将 VM 标记为通用。

    ```powershell
    Set-AzVm -ResourceGroupName $rgName -Name $vmName -Generalized  
    ```
4. 使用通用化的 OS VHD 创建映像。

    ```powershell
    $imageConfig = New-AzImageConfig -Location $location
    $imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsType Windows -OsState Generalized -BlobUri $osVhdUri
    $image = New-AzImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ```

## <a name="next-steps"></a>后续步骤
- [从托管的映像创建 VM](create-vm-generalized-managed.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)。

<!-- Update_Description: update meta properties, wording update, update link -->