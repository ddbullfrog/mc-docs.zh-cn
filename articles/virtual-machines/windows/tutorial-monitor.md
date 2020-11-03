---
title: 教程 - 监视 Azure 中的 Windows 虚拟机
description: 本教程介绍如何监视 Windows 虚拟机上运行的性能和发现的应用程序组件。
manager: carmonm
ms.service: virtual-machines-windows
ms.subservice: monitoring
ms.topic: tutorial
ms.workload: infrastructure
origin.date: 09/27/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: 82d50f9f942b717817901c98dc80099d57ffdad1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105782"
---
# <a name="tutorial-monitor-a-windows-virtual-machine-in-azure"></a>教程：监视 Azure 中的 Windows 虚拟机

Azure 监视使用代理从 Azure VM 收集启动和性能数据，将此数据存储在 Azure 存储中，并使其可供通过门户、Azure PowerShell 模块和 Azure CLI 进行访问。 用于 VM 的 Azure Monitor 通过收集性能指标、发现 VM 上安装的应用程序组件来提供高级监视，并包括性能图表和依赖关系图。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 在 VM 上启用启动诊断
> * 查看启动诊断
> * 查看 VM 主机指标
> * 创建警报

<!--Not Available on > * Enable Azure Monitor for VMs-->
<!--Not Available on > * View VM performance metrics-->

## <a name="launch-azure-local-shell"></a>启动 Azure 本地 Shell

打开 Azure Powershell 控制台，以管理员权限运行下面列出的脚本。

<!--Not Available on Azure Cloud Shell-->

## <a name="create-virtual-machine"></a>创建虚拟机

若要在本教程中配置 Azure 监视和更新管理，需要 Azure 中的 Windows VM。 首先，使用 [Get-Credential](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) 设置 VM 的管理员用户名和密码：

```powershell
$cred = Get-Credential
```

现在，使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 以下示例在“ChinaEast”位置创建一个名为 myVM 的 VM。 如果资源组 *myResourceGroupMonitorMonitor* 和支持的网络资源不存在，则会创建它们：

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "China East" `
    -Credential $cred
```

创建资源和 VM 需要几分钟的时间。

## <a name="view-boot-diagnostics"></a>查看启动诊断

当 Windows 虚拟机启动时，启动诊断代理将捕获屏幕输出，可以使用该输出进行故障排除。 此功能是默认启用的。 捕获的屏幕截图存储在一个 Azure 存储帐户中，该帐户也是默认创建的。

可以使用 [Get-AzureRmVMBootDiagnosticsData](https://docs.microsoft.com/powershell/module/az.compute/get-azvmbootdiagnosticsdata) 命令获取启动诊断数据。 在下面的示例中，启动诊断下载到了 *c:\* 驱动器的根目录中。

```powershell
Get-AzVMBootDiagnosticsData -ResourceGroupName "myResourceGroupMonitor" -Name "myVM" -Windows -LocalPath "c:\"
```

## <a name="view-host-metrics"></a>查看主机指标

Windows VM 在 Azure 中有一个与它交互的专用主机 VM。 系统会自动收集该主机的指标，可以在 Azure 门户中查看这些指标。

1. 在 Azure 门户中单击“资源组”，选择“myResourceGroupMonitor”，并在资源列表中选择“myVM”。
2. 要查看主机 VM 的性能情况，请在 VM 边栏选项卡上单击“指标”，并选择“可用指标”下的任一主机指标。

    :::image type="content" source="./media/tutorial-monitoring/tutorial-monitor-host-metrics.png" alt-text="查看主机指标":::

<!--Not Available on ## Enable advanced monitoring-->
<!--Not Available on  **Insights (preview)** till 08/24/2020-->

<!--Not Available on ## View VM performance metrics-->

<!--Not Available on  **Insights (preview)**-->

## <a name="create-alerts"></a>创建警报

可以根据特定的性能指标创建警报。 例如，当平均 CPU 使用率超过特定的阈值或者可用磁盘空间低于特定的空间量时，警报可用于发出通知。 警报显示在 Azure 门户中，也可以通过电子邮件发送。 还可以触发 Azure 自动化 Runbook 或 Azure 逻辑应用来响应生成的警报。

以下示例针对平均 CPU 使用率创建警报。

1. 在 Azure 门户中单击“资源组”，选择“myResourceGroupMonitor”，并在资源列表中选择“myVM”。

2. 在 VM 边栏选项卡上单击“警报规则”，然后单击警报边栏选项卡顶部的“添加指标警报”。

3. 为警报提供 **名称** ，例如 *myAlertRule*

4. 若要在 CPU 百分比持续 5 分钟超过 1.0 时触发警报，请选中其他所有默认值。

5. （可选）选中“电子邮件所有者、参与者和读者”对应的框，以便向他们发送电子邮件通知。 默认操作是在门户中显示通知。

6. 单击“确定”按钮。

## <a name="next-steps"></a>后续步骤

在本教程中，你配置并查看了 VM 的性能。 你已了解如何：

> [!div class="checklist"]
> * 创建资源组和 VM
> * 在 VM 上启用启动诊断
> * 查看启动诊断
> * 查看主机指标
> * 创建警报

<!--Not Available on > * Enable Azure Monitor for VMs-->
<!--Not Available on > * View VM performance metrics-->

请转到下一教程来了解 Azure 安全中心。

> [!div class="nextstepaction"]
> [管理 VM 安全性](../../security/fundamentals/overview.md)

<!-- Update_Description: update meta properties, wording update, update link -->
