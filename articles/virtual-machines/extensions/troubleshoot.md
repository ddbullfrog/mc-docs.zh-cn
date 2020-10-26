---
title: Windows VM 扩展故障排除
description: 了解如何进行 Azure Windows VM 扩展故障排除
services: virtual-machines-windows
documentationcenter: ''
manager: gwallace
editor: ''
tags: top-support-issue,azure-resource-manager
ms.assetid: 878ab9b6-c3e6-40be-82d4-d77fecd5030f
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
origin.date: 03/29/2016
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 10/19/2020
ms.author: v-yeche
ms.openlocfilehash: af999cf937f89673dcfdfd39313e512c51f3274c
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128067"
---
# <a name="troubleshooting-azure-windows-vm-extension-failures"></a>Azure Windows VM 扩展故障排除
[!INCLUDE [virtual-machines-common-extensions-troubleshoot](../../../includes/virtual-machines-common-extensions-troubleshoot.md)]

## <a name="viewing-extension-status"></a>查看扩展状态
可以从 Azure PowerShell 执行 Azure 资源管理器模板。 一旦执行该模板，就可以从 Azure 资源浏览器或命令行工具查看扩展状态。

以下是示例：

Azure PowerShell：

```azurepowershell
Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status
```

下面是示例输出：

```output
Extensions:  {
  "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
  "Name": "myCustomScriptExtension",
  "SubStatuses": [
    {
      "Code": "ComponentStatus/StdOut/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
          \\n---- -------------     ------ ----                              \\n-a--- 9/1/2015   2:03 AM         11
          test.txt                          \\n\\n",
                  "Time": null
      },
    {
      "Code": "ComponentStatus/StdErr/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "",
      "Time": null
    }
  ]
}
```

## <a name="troubleshooting-extension-failures"></a>扩展故障排除

### <a name="rerun-the-extension-on-the-vm"></a>在 VM 上重新运行扩展
如果使用自定义脚本扩展在 VM 上运行脚本，有时可能会遇到错误：VM 已成功创建但脚本却运行失败。 在这些情况下，从此错误中恢复的建议方法是删除该扩展并再次重新运行该模板。
注意：未来此功能将得到增强，不再需要卸载该扩展。

#### <a name="remove-the-extension-from-azure-powershell"></a>从 Azure PowerShell 删除扩展
```azurepowershell
Remove-AzVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"
```

删除该扩展后，可以重新执行模板在 VM 上运行脚本。

### <a name="trigger-a-new-goalstate-to-the-vm"></a>触发 VM 的新 GoalState
你可能会注意到某个扩展尚未执行，或者由于缺少“Windows Azure CRP 证书生成器”（该证书用于保护扩展的受保护设置的传输）而无法执行。
可以通过从虚拟机内重启 Windows 来宾代理自动重新生成该证书：
- 打开任务管理器
- 转到“详细信息”选项卡
- 找到 WindowsAzureGuestAgent.exe 进程
- 右键单击并选择“结束任务”。 该进程将自动重启

还可以通过执行“空更新”来触发 VM 的新 GoalState：

Azure PowerShell：

```azurepowershell
$vm = Get-AzureRMVM -ResourceGroupName <RGName> -Name <VMName>  
Update-AzureRmVM -ResourceGroupName <RGName> -VM $vm  
```

Azure CLI：

```azurecli
az vm update -g <rgname> -n <vmname>
```

如果“空更新”不起作用，则可以从 Azure 管理门户向 VM 添加新的空数据磁盘，然后在重新添加证书后将其删除。

<!-- Update_Description: update meta properties, wording update, update link -->