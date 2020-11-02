---
title: 使用远程工具排查 Azure VM 问题 | Azure
description: 了解 PsExec、PowerShell 脚本和其他可用来在不使用 RDP 的情况下排查远程 Azure VM 问题的远程工具。
services: virtual-machines-windows
manager: dcscontentpm
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
origin.date: 01/11/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: c2f00ef127b088ef96b1a3edbc91a07904fc98bc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105416"
---
# <a name="use-remote-tools-to-troubleshoot-azure-vm-issues"></a>使用远程工具排查 Azure VM 问题

排查 Azure 虚拟机 (VM) 的问题时，可以使用本文所述的远程工具而不是远程桌面协议 (RDP) 连接到 VM。

<!--Not Available on ## Serial Console-->

<!--Not Available on Use [Virtual Machine Serial Console](serial-console-windows.md)-->

## <a name="remote-cmd"></a>远程 CMD

下载 [PsExec](https://docs.microsoft.com/sysinternals/downloads/psexec)。 运行以下命令连接到 VM：

```cmd
psexec \\<computer> -u user -s cmd
```

<!--MOONCAKE: Correct on \\<computer> -u-->

>[!NOTE]
>* 必须在位于同一虚拟网络中的计算机上运行该命令。
>* 可以使用 DIP 或主机名来替换 \<computer>。
>* -s 参数确保使用系统帐户（管理员权限）调用命令。
>* PsExec 使用 TCP 端口 135 和 445。 因此，需要在防火墙中打开这两个端口。

## <a name="run-command"></a>运行命令

有关如何使用“运行命令”功能在 VM 上运行脚本的详细信息，请参阅[使用“运行命令”在 Windows VM 中运行 PowerShell 脚本](../windows/run-command.md)。

## <a name="custom-script-extension"></a>自定义脚本扩展

可以使用“自定义脚本扩展”功能在目标 VM 上运行自定义脚本。 若要使用此功能，必须符合以下条件：

* VM 已建立连接。
* 已在 VM 上安装 Azure 虚拟机代理，并且该代理正在按预期方式运行。
* 未事先在 VM 上安装该扩展。

    该扩展仅在首次使用时才注入脚本。 如果以后再使用此功能，该扩展会意识到它已被用过，因此不会上传新脚本。

将脚本上传到一个存储帐户，并生成其自己的容器。 然后，在已连接到 VM 的计算机上的 Azure PowerShell 中运行以下脚本。

### <a name="for-classic-deployment-model-vms"></a>对于经典部署模型 VM

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

```powershell
#Set up the basic variables.
$subscriptionID = "<<SUBSCRIPTION ID>>" 
$storageAccount = "<<STORAGE ACCOUNT>>" 
$localScript = "<<FULL PATH OF THE PS1 FILE TO EXECUTE ON THE VM>>" 
$blobName = "file.ps1" #Name you want for the blob in the storage.
$vmName = "<<VM NAME>>" 
$vmCloudService = "<<CLOUD SERVICE>>" #Resource group or cloud service where the VM is hosted. For example, for "demo305.chinacloudapp.cn" the cloud service is going to be demo305.

#Set up the Azure PowerShell module, and ensure the access to the subscription.
Import-Module Azure
Add-AzureAccount -Environment AzureChinaCloud  #Ensure login with the account associated with the subscription ID.
Get-AzureSubscription -SubscriptionId $subscriptionID | Select-AzureSubscription

#Set up the access to the storage account, and upload the script.
$storageKey = (Get-AzureStorageKey -StorageAccountName $storageAccount).Primary
$context = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccount -StorageAccountKey $storageKey
$container = "cse" + (Get-Date -Format yyyyMMddhhmmss)<
New-AzureStorageContainer -Name $container -Permission Off -Context $context
Set-AzureStorageBlobContent -File $localScript -Container $container -Blob $blobName  -Context $context

#Push the script into the VM.
$vm = Get-AzureVM -ServiceName $vmCloudService -Name $vmName
Set-AzureVMCustomScriptExtension "CustomScriptExtension" -VM $vm -StorageAccountName $storageAccount -StorageAccountKey $storagekey -ContainerName $container -FileName $blobName -Run $blobName | Update-AzureVM
```

### <a name="for-azure-resource-manager-vms"></a>对于 Azure 资源管理器 VM

```powershell
#Set up the basic variables.
$subscriptionID = "<<SUBSCRIPTION ID>>"
$storageAccount = "<<STORAGE ACCOUNT>>"
$storageRG = "<<RESOURCE GROUP OF THE STORAGE ACCOUNT>>" 
$localScript = "<<FULL PATH OF THE PS1 FILE TO EXECUTE ON THE VM>>" 
$blobName = "file.ps1" #Name you want for the blob in the storage.
$vmName = "<<VM NAME>>" 
$vmResourceGroup = "<<RESOURCE GROUP>>"
$vmLocation = "<<DATACENTER>>" 

#Set up the Azure PowerShell module, and ensure the access to the subscription.
Connect-AzAccount -Environment AzureChinaCloud #Ensure login with the account associated with the subscription ID.
Get-AzSubscription -SubscriptionId $subscriptionID | Select-AzSubscription

#Set up the access to the storage account, and upload the script.
$storageKey = (Get-AzStorageAccountKey -ResourceGroupName $storageRG -Name $storageAccount).Value[0]
$context = New-AzStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccount -StorageAccountKey $storageKey
$container = "cse" + (Get-Date -Format yyyyMMddhhmmss)
New-AzStorageContainer -Name $container -Permission Off -Context $context
Set-AzStorageBlobContent -File $localScript -Container $container -Blob $blobName -Context $context

#Push the script into the VM.
Set-AzVMCustomScriptExtension -Name "CustomScriptExtension" -ResourceGroupName $vmResourceGroup -VMName $vmName -Location $vmLocation -StorageAccountName $storageAccount -StorageAccountKey $storagekey -ContainerName $container -FileName $blobName -Run $blobName
```

<!--Not Available on AzureStorage: The term 'New-AzureStorageContainer' is not recognized as the name of a cmdlet-->

<!--Not Available on ## Remote PowerShell-->
## <a name="remote-registry"></a>远程注册表

>[!NOTE]
>必须打开 TCP 端口 135 或 445 才能使用此选项。
>
>对于 Azure 资源管理器 VM，必须在 NSG 上打开端口 5986。 有关详细信息，请参阅“安全组”。 
>
>对于 RDFE VM，必须有一个配备专用端口 5986 和公共端口的终结点。 还必须在 NSG 中打开该公共端口。

1. 在同一虚拟网络中的另一个 VM 上，打开注册表编辑器 (regedit.exe)。

2. 选择“文件” > “连接网络注册表”。  

    :::image type="content" source="./media/remote-tools-troubleshoot-azure-vm-issues/remote-registry.png" alt-text="注册表编辑器"::: 

3. 在“输入要选择的对象名称”框中输入目标 VM 的主机名或动态 IP（首选），以找到该 VM。   

    :::image type="content" source="./media/remote-tools-troubleshoot-azure-vm-issues/input-computer-name.png" alt-text="注册表编辑器"::: 

4. 输入目标 VM 的凭据。

5. 进行任何必要的注册表更改。

## <a name="remote-services-console"></a>远程服务控制台

>[!NOTE]
>必须打开 TCP 端口 135 或 445 才能使用此选项。
>
>对于 Azure 资源管理器 VM，必须在 NSG 上打开端口 5986。 有关详细信息，请参阅“安全组”。 
>
>对于 RDFE VM，必须有一个配备专用端口 5986 和公共端口的终结点。 还必须在 NSG 中打开该公共端口。

1. 在同一虚拟网络中的另一个 VM 上，打开 Services.msc  的一个实例。

2. 右键单击“服务(本地)”。 

3. 选择“连接到另一台计算机”。 

    :::image type="content" source="./media/remote-tools-troubleshoot-azure-vm-issues/remote-services.png" alt-text="注册表编辑器":::

4. 输入目标 VM 的动态 IP。

    :::image type="content" source="./media/remote-tools-troubleshoot-azure-vm-issues/input-ip-address.png" alt-text="注册表编辑器":::

5. 对服务进行任何必要的更改。

## <a name="next-steps"></a>后续步骤

- 有关 Enter-PSSession cmdlet 的详细信息，请参阅 [Enter-PSSession](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-5.1)。
- 若要详细了解使用经典部署模型完成的适用于 Windows 的自定义脚本扩展，请参阅[适用于 Windows 的自定义脚本扩展](../extensions/custom-script-windows.md)。
- PsExec 包含在 [PSTools Suite](https://download.sysinternals.com/files/PSTools.zip) 中。
- 有关 PSTools Suite 的详细信息，请参阅 [PSTools](https://docs.microsoft.com/sysinternals/downloads/pstools)。

<!-- Update_Description: update meta properties, wording update, update link -->