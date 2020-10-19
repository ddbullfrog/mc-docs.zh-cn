---
title: 如何在 Windows 或 Linux 虚拟机和虚拟机规模集上升级 Azure 磁盘加密扩展 | Microsoft Docs
description: 如何在 Windows 或 Linux 虚拟机和虚拟机规模集上升级 Azure 磁盘加密扩展。
services: na
documentationcenter: na
editor: ''
author: ''
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 10/10/2020
ms.date: 10/10/2020
ms.author: v-yiso
ms.openlocfilehash: c955c8b92318dfe21ac4a488f2cff4be1bf79d48
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937670"
---
# <a name="how-to-upgrade-your-azure-disk-encryption-extension-on-your-windows-or-linux-virtual-machines-and-virtual-machine-scale-sets"></a>如何在 Windows 或 Linux 虚拟机和虚拟机规模集上升级 Azure 磁盘加密扩展

## <a name="a---scope"></a>A.   范围
此更改的范围仅限于不带 AAD 凭据的 Azure 磁盘加密扩展版本。 若要更新到最新的扩展版本，需要执行以下操作。 下面的 B 和 C 部分包含了详细步骤。
 1. 识别使用不带 AAD 凭据的 Azure 磁盘加密扩展（通常称为单次传递 ADE）的虚拟机或虚拟机规模集。
 2. 运行此操作以执行目标状态更新，将 Azure 磁盘加密扩展更新到最新版本。

**如果 VM 或规模集使用带 AAD 凭据的 Azure 磁盘加密（通常称为双重传递 ADE），则无需执行任何操作。** 

**受影响的扩展版本：**
- Windows:版本 2.2.0.0 到 2.2.0.34
- Linux：  版本 1.1.0.0 到 1.1.0.51

## <a name="b---determine-ade-extension-version"></a>B.   确定 ADE 扩展版本 

### <a name="virtual-machines"></a>虚拟机

#### <a name="using-azure-cli"></a>使用 Azure CLI
1. 在 Azure CLI 提示符下运行以下命令。 ADE 扩展将在输出中作为“extensions”节的一部分显示。
```bash
az vm get-instance-view --resource-group  <ResourceGroupName> --vm-name <VMName> 
```
2.  找到输出中的 AzureDiskEncryption 扩展，从输出中的“TypeHandlerVersion”字段识别版本号。
 
#### <a name="using-powershell"></a>使用 PowerShell
1.  在 PowerShell 提示符下运行以下命令。 ADE 扩展将在输出中显示为“Extensions[]”节的一部分。
```powershell
Get-AzVM -ResourceGroupName <ResourceGroupName> -Name <VMName> -Status
```
2.  找到输出中的 AzureDiskEncryption 扩展，从输出中的“TypeHandlerVersion”字段识别版本号。 

#### <a name="using-the-azure-portal"></a>使用 Azure 门户
1.  在 Azure 门户中转到 VM 的“扩展”边栏选项卡。 

![ExtensionsBlade](https://user-images.githubusercontent.com/55103122/93830258-9100ea80-fc24-11ea-9e71-35a1cc795e18.png)

2.  选择适用于 Windows 的“AzureDiskEncryption”扩展或适用于 Linux 的“AzureDiskEncryptionForLinux”扩展。 
3.  在“版本”字段中找到版本号。 

![版本](https://user-images.githubusercontent.com/55103122/93830268-978f6200-fc24-11ea-8141-a7671b02a24b.jpg)

### <a name="virtual-machine-scale-sets"></a>虚拟机规模集
#### <a name="using-azure-cli"></a>使用 Azure CLI
1. 在 Azure CLI 提示符下运行以下命令。 --instance-id 参数应该是规模集中存在的某一 VM 实例的有效实例编号。ADE 扩展将在输出中显示为“extensions”节的一部分。
```bash
az vmss get-instance-view --resource-group  <ResourceGroupName> --name <ScaleSet_Name> --instance-id <VMSS_InstanceId>
```
2.  找到输出中的 AzureDiskEncryption 扩展，从输出中的“TypeHandlerVersion”字段识别版本号。

#### <a name="using-powershell"></a>使用 PowerShell
1.  在 PowerShell 提示符下运行以下命令。 “InstanceId”参数应该是规模集中存在的某一 VM 实例的有效实例编号。 ADE 扩展将显示为输出的“Extensions[]”节的一部分。
```powershell
Get-AzVMSSVM -ResourceGroupName <ResourceGroupName> - VMScaleSetName <ScaleSet_Name> -InstanceId <InstanceNumber> -InstanceView 
```
2.  在输出的“ExtensionProfile”字段中找到 AzureDiskEncryption 扩展。 
3.  从输出中的“TypeHandlerVersion”字段识别版本号。

#### <a name="using-the-azure-portal"></a>使用 Azure 门户
1.  在 Azure 门户中转到规模集的“扩展”边栏选项卡。 

![ExtensionsBladeVMSS](https://user-images.githubusercontent.com/55103122/93830276-9f4f0680-fc24-11ea-8d18-d9f905acfa86.png)

2.  从“扩展”边栏选项卡选择适用于 Windows 的“AzureDiskEncryption”扩展或适用于 Linux 的“AzureDiskEncryptionForLinux”扩展。 
3.  在“版本”字段中找到版本号。 

![VMSSVersion](https://user-images.githubusercontent.com/55103122/93830283-a37b2400-fc24-11ea-9125-742796be3f6b.jpg)

## <a name="c---update-the-ade-extension-to-the-newest-version"></a>C.   将 ADE 扩展更新到最新版本
> 扩展更新将于 2020 年 10 月 6 日推出

若要下载包含变更的 Azure 磁盘加密扩展的最新版本，需要在 VM/VMSS 中触发目标状态更新。 可以应用以下任一选项来触发目标状态更新。 
### <a name="prerequisites"></a>先决条件
1.  确保 Azure Windows VM 代理/Azure Linux VM 代理正在 VM/VMSS 中运行。 
> 注意：[VM 代理存在最低支持版本](https://docs.microsoft.com/en-us/troubleshoot/azure/virtual-machines/support-extensions-agent-version)。
2.  Azure 市场映像装有 VM 代理。 如果手动卸载了 VM 代理，请按照以下链接中的说明安装适用于 Windows 或 Linux 的 VM 代理。
    - [Windows VM 代理文档](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/agent-windows)
    - [Linux VM 代理文档](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/agent-linux)

### <a name="option-1-reboot-recommended"></a>选项 1：重启（建议）
1.  确定受影响的计算机的维护时段。 
2.  重启 VM 或规模集实例，以便触发目标状态更新，从而在计算机上下载最新版本的扩展。
#### <a name="example-1-the-below-code-snippet-restarts-all-the-virtual-machines-present-in-a-resource-group"></a>示例 1：以下代码片段会重启资源组中存在的所有虚拟机
```powershell
#Replace "rgname" with your resource group name
$ResourceGroupName = "rgname" 

Get-AzVM -ResourceGroupName $ResourceGroupName | Select Name | ForEach-Object {

    Write-Host "Restarting virtual machine: $_.Name"
    Restart-AzVM -ResourceGroupName $ResourceGroupName -Name $_.Name

}
```

#### <a name="example-2-the-below-code-snippet-restarts-virtual-machines-specified-in-the-variable-virtualmachinelist-note-that-all-these-virtual-machines-must-be-part-of-the-same-resource-group"></a>示例 2：以下代码片段会重启变量“virtualMachineList”中指定的虚拟机。 请注意，所有这些虚拟机都必须属于同一个资源组 
```powershell
#Replace "rgname" with your resource group name
$ResourceGroupName = "rgname" 

#Replace "vmName" with your virtual machine names. Separate vmNames with a comma when adding more vmNames. 
$virtualMachinesList = @("vmName1", "vmName2", "vmName3")
foreach ($vmName in $virtualMachinesList)
{    
    Write-Host "Restarting virtual machine: $vmName"
    Restart-AzVM -ResourceGroupName $ResourceGroupName -Name $vmName -Verbose
}
```
> 对于虚拟机，请转到“D 部分：验证”。

> 对于虚拟机规模集，请执行下面的 2 个额外步骤。
3.  检查“升级策略”是“手动”还是“滚动”。 
 - #### <a name="azure-powershell"></a>Azure PowerShell
   - 此命令输出在创建 VMSS 时指定的升级策略。
```powershell
$LocalVMSS = Get-AzVMSS -ResourceGroupName <ResourceGroupName> - VMScaleSetName <ScaleSet_Name>
```
- #### <a name="azure-portal"></a>Azure 门户
  - 在 VMSS 资源中，转到“升级策略”。

  - ![VMSSUpgradePolicy](https://user-images.githubusercontent.com/55103122/93830288-a70eab00-fc24-11ea-8591-e88c15fd45a0.jpg)

4.  如果步骤 3 适用，请按以下步骤将 VMSS 实例更新为最新的 VMSS VM 模型。 
>注意：对于 VMSS，当多个扩展插件与规模集相关联时，Azure 磁盘加密需要扩展排序。 请参阅：[磁盘加密扩展排序](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/disk-encryption-extension-sequencing)。
 - #### <a name="azure-powershell"></a>Azure PowerShell
 ```powershell
 $LocalVMSS = Get-AzVMSS -ResourceGroupName <ResourceGroupName> - VMScaleSetName <ScaleSet_Name>
 Update-AzVMSS -ResourceGroupName <ResourceGroupName> -Name <ScaleSet_Name> -VirtualMachineScaleSet $LocalVMSS
 ```
 - #### <a name="azure-cli"></a>Azure CLI
```bash
az vmss update-instances --instance-ids * --name <ScaleSet_Name> --resource-group <ResourceGroupName>
```
 - #### <a name="azure-portal"></a>Azure 门户
  1.    在 Azure 门户中转到 VMSS 的“实例”边栏选项卡。 
  2.    选择所有实例，然后单击“升级”。 
  3.    请确保升级后所有实例的“最新模型”字段更改为“是”。

  ![VMSSLatestModel](https://user-images.githubusercontent.com/55103122/93830293-a8d86e80-fc24-11ea-8537-3b43ea953bce.png)

### <a name="option-2-run-command-feature"></a>选项 2：“运行命令”功能
若要避免重启，请使用 VM 代理的“运行命令”功能。 这是应在 VM 外部运行的远程命令。  此命令通过 VM 代理触发 VM 上的目标状态更新。 
1.  请参阅提供的链接，查看相关选项，了解如何针对 VM 和 VMSS 使用 VM 代理的“运行命令”功能。 
> **请使用“运行命令”功能运行一个不会中断虚拟机的运行时状态的命令** 
> - **对于 Windows：IPConfig 或 RDPSetting**
> - **对于 Linux：ifconfig**
- #### <a name="azure-powershell"></a>Azure PowerShell 
  - [Windows 运行命令 (PowerShell)](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/run-command#powershell)
  - [Linux 运行命令 (PowerShell)](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/run-command#powershell) 
 - #### <a name="azure-cli"></a>Azure CLI 
   - [Windows 运行命令 (Azure CLI)](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/run-command#azure-cli)
   - [Linux 运行命令 (Azure CLI)](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/run-command#azure-cli) 
 - #### <a name="azure-portal"></a>Azure 门户 
   - [Windows 运行命令（Azure 门户）](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/run-command#azure-portal)
   - [Linux 运行命令（Azure 门户）](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/run-command#azure-portal) 
> 对于虚拟机，请转到“D 部分：验证”。

> 对于虚拟机规模集，请执行下面的 2 个额外步骤。
2.  检查“升级策略”是“手动”还是“滚动”。 
 - #### <a name="azure-powershell"></a>Azure PowerShell
   - 此命令输出在创建 VMSS 时指定的升级策略。
```powershell
$LocalVMSS = Get-AzVMSS -ResourceGroupName <ResourceGroupName> - VMScaleSetName <ScaleSet_Name>
```
- #### <a name="azure-portal"></a>Azure 门户
  - 在 VMSS 资源中，转到“升级策略”。

  - ![VMSSUpgradePolicy](https://user-images.githubusercontent.com/55103122/93830288-a70eab00-fc24-11ea-8591-e88c15fd45a0.jpg)

3.  如果步骤 2 适用，请按以下步骤将 VMSS 实例更新为最新的 VMSS VM 模型。 
>注意：对于 VMSS，当多个扩展插件与规模集相关联时，Azure 磁盘加密需要扩展排序。 请参阅：[磁盘加密扩展排序](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/disk-encryption-extension-sequencing)。
 - #### <a name="azure-powershell"></a>Azure PowerShell
 ```powershell
 $LocalVMSS = Get-AzVMSS -ResourceGroupName <ResourceGroupName> - VMScaleSetName <ScaleSet_Name>
 Update-AzVMSS -ResourceGroupName <ResourceGroupName> -Name <ScaleSet_Name> -VirtualMachineScaleSet $LocalVMSS
 ```
 - #### <a name="azure-cli"></a>Azure CLI
```bash
az vmss update-instances --instance-ids * --name <ScaleSet_Name> --resource-group <ResourceGroupName>
```
 - #### <a name="azure-portal"></a>Azure 门户
  1.    在 Azure 门户中转到 VMSS 的“实例”边栏选项卡。 
  2.    选择所有实例，然后单击“升级”。 
  3.    请确保升级后所有实例的“最新模型”字段更改为“是”。

  ![VMSSLatestModel](https://user-images.githubusercontent.com/55103122/93830293-a8d86e80-fc24-11ea-8537-3b43ea953bce.png)

## <a name="d-validation"></a>D. 验证
在 VM/VMSS 中完成目标状态更新后，验证是否已将 Azure 磁盘加密扩展更新为以下版本。 
> 从 C 部分触发目标状态更新后，请至少等待 5 分钟
1.  运行“B 部分：确定 ADE 扩展版本”中引用的脚本，以便验证 Windows 和 Linux 的扩展版本 
    - Windows:2.2.0.35 及更高版本
    - Linux：  1.1.0.52 及更高版本

## <a name="e-faq"></a>E. 常见问题解答
1.  我的 VM/VMSS 通过 ADE 进行加密，但 ADE 扩展已从 VM/VMSS 中删除。 是否应再次安装 ADE？ 
> 能。 如果已从 VM/VMSS 中删除该扩展，请按照首次启用 ADE 时所使用的步骤进行操作。 这样就会将 ADE 扩展的最新版本添加回资源。 注意：对于 VMSS，当多个扩展插件与规模集相关联时，Azure 磁盘加密需要扩展排序。 请参阅：磁盘加密扩展排序
2.  已在 VM/VMSS 上禁用了 ADE。 是否仍然应当应用缓解步骤？ 
> 目标状态更新会将禁用的 ADE 扩展更新到最新版本。 注意：当 ADE 处于禁用状态时，不会从 VM 中删除 ADE 扩展。