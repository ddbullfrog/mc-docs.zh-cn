---
title: 教程 - 在 Azure 中管理 Windows 虚拟机配置
description: 在本教程中，你将学习如何在 Windows 虚拟机上识别更改和管理包更新版
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
origin.date: 12/05/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: 5a14e3fc92de35202ddafe639e7047d6518efc01
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104994"
---
<!--Verified successfully-->
# <a name="tutorial-monitor-changes-and-update-a-windows-virtual-machine-in-azure"></a>教程：监视更改和更新 Azure 中的 Windows 虚拟机

借助 Azure [更新管理](../../automation/update-management/update-mgmt-overview.md)，可以轻松地识别 Azure 中 Windows 虚拟机中的更改并管理这些 VM 的操作系统更新。

<!--Not Available on [Change Tracking](../../automation/change-tracking.md)-->

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 管理 Windows 更新。
> * 监视器更改和清单。

## <a name="open-azure-local-shell"></a>打开 Azure 本地 Shell

打开 Azure Powershell 控制台，并以管理员权限运行以下脚本。

<!--Not Available on Azure Cloud Shell-->
<!--Not Available on [https://shell.azure.com/powershell](https://shell.azure.com/powershell)-->

## <a name="create-a-virtual-machine"></a>创建虚拟机

若要在本教程中配置 Azure 监视和更新管理，需要 Azure 中的 Windows VM。

首先，使用 [Get-Credential](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) 设置 VM 的管理员用户名和密码：

```powershell
$cred = Get-Credential
```

接下来，使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 以下示例在 `China East` 位置创建名为 `myVM` 的 VM。 如果资源组 `myResourceGroupMonitor` 和支持的网络资源尚不存在，则会创建它们：

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "China East" `
    -Credential $cred
```

创建资源和 VM 需要几分钟的时间。

## <a name="manage-windows-updates"></a>管理 Windows 更新

使用更新管理有助于管理 Azure Windows VM 的更新和修补程序。 直接从 VM 可以快速执行以下操作：

- 评估可用更新的状态。
- 计划安装所需的更新。
- 查看部署结果，验证更新是否已成功应用到 VM。

有关定价信息，请参阅[更新管理的自动化定价](https://www.azure.cn/pricing/details/automation/)。

### <a name="enable-update-management"></a>启用更新管理

若要为 VM 启用更新管理：

1. 请在窗口的最左侧，选择“虚拟机”。
1. 从列表中选择 VM。
1. 在 VM 窗口的“操作”窗格中，选择“更新管理” 。
1. “启用更新管理”窗口随即打开。

执行验证以确定是否为该 VM 启用了更新管理。 验证包括检查 Log Analytics 工作区和链接的自动化帐户，以及解决方案是否在工作区中。

应使用 [Log Analytics](../../azure-monitor/log-query/log-query-overview.md) 工作区收集由功能和服务（如更新管理）生成的数据。 工作区提供了一个位置来查看和分析来自多个数据源的数据。

若要在需要更新的 VM 上执行其他操作，可使用 Azure 自动化针对 VM 运行 Runbook。 此类操作包括下载或应用更新。

验证过程还会检查 VM 是否预配了 Microsoft Monitoring Agent (MMA) 和自动化混合 Runbook 辅助角色。 使用此代理与虚拟机通信并获取关于更新状态的信息。

<!--CORRECT ON Microsoft Monitoring Agent (MMA)-->

在“启用更新管理”窗口中，选择 Log Analytics 工作区和自动化帐户，然后选择“启用” 。 启用此解决方案最长需要 15 分钟的时间。

加入过程中缺少的下列任何先决条件会自动添加：

* [Log Analytics](../../azure-monitor/log-query/log-query-overview.md) 工作区
* [自动化](../../automation/index.yml)
* [混合 runbook 辅助角色](../../automation/automation-hybrid-runbook-worker.md)，VM 上已启用此辅助角色

启用解决方案后，将打开“更新管理”窗口。 配置要使用的位置、Log Analytics 工作区和自动化帐户，然后选择“启用”。 如果这些字段变暗，则意味着已为 VM 启用其他自动化解决方案，并且必须使用该解决方案工作区和自动化帐户。

:::image type="content" source="./media/tutorial-monitoring/manageupdates-update-enable.png" alt-text="启用更新管理解决方案":::

可能最多需要 15 分钟才能启用更新管理解决方案。 在此期间，请勿关闭浏览器窗口。 启用该解决方案后，VM 中缺少的更新信息会流向 Azure Monitor 日志。 可能需要 30 分钟至 6 小时才可将数据用于分析。

### <a name="view-an-update-assessment"></a>查看更新评估

启用“更新管理”后，“更新管理”窗口随即显示。 评估更新完成后，可在“缺失更新”选项卡上查看缺失更新的列表。

:::image type="content" source="./media/tutorial-monitoring/manageupdates-view-status-win.png" alt-text="启用更新管理解决方案":::

### <a name="schedule-an-update-deployment"></a>计划更新部署

若要安装更新，请计划一个遵循你的发布时间和服务窗口的部署。 可选择在部署中包括哪种更新类型。 例如，可包括关键或安全更新，排除更新汇总。

若要为 VM 计划新的更新部署，请选择“更新管理”窗口顶部的“计划更新部署”。 在“新建更新部署”窗口中，指定以下信息：

| 选项 | 说明 |
| --- | --- |
| **名称** |输入用于标识更新部署的唯一名称。 |
|**操作系统**| 选择 Linux 或 Windows 。|
| **要更新的组** |对于在 Azure 上托管的 VM，应基于订阅、资源组、位置和标记的组合定义查询。 此查询将生成要包含在你的部署中的 Azure 托管 VM 的动态组。 <br /><br />对于未托管在 Azure 上的 VM，请选择现有的已保存搜索。 使用此搜索，可以选择要包括在部署中的一组 VM。 <br /><br /> 有关详细信息，请参阅[动态组](../../automation/update-management/update-mgmt-groups.md)。|
| **要更新的计算机** |请选择“已保存的搜索”、“已导入的组”或“计算机”  。<br/><br/>如果选择“计算机”，则可以从下拉列表中选择单个计算机。 每台计算机的准备情况将显示在表的“更新代理准备情况”列中。<br /><br /> 要了解在 Azure Monitor 日志中创建计算机组的不同方法，请参阅 [Azure Monitor 日志中的计算机组](../../azure-monitor/platform/computer-groups.md) |
|**更新分类**|选择所有必需的更新分类。|
|**包括/排除更新**|选择此选项可打开“包括/排除”窗格。 要包含或排除的更新位于单独的选项卡上。 有关如何处理包含的详细信息，请参阅[计划更新部署](../../automation/update-management/update-mgmt-deploy-updates.md#schedule-an-update-deployment)。 |
|**计划设置**|选择启动时间，然后选择“一次”或“定期” 。|
| **前脚本 + 后脚本**|选择要在部署前和部署后运行的脚本。|
| **维护时段** | 输入为更新设置的分钟数。 有效值范围为 30 至 360 分钟。 |
| **重新启动控制**| 选择处理重启的方式。 可用选项包括：<ul><li>**必要时请重启**</li><li>**永远重启**</li><li>**永不重启**</li><li>**仅重启**</li></ul>默认选项为“必要时重启”。 若选择“仅重启”，则不会安装更新。|

完成配置计划后，单击“创建”可返回到状态仪表板。 “已计划”表会显示你创建的部署计划。

还能够以编程方式创建更新部署。 若要了解如何使用 REST API 创建更新部署，请参阅[软件更新配置 - 创建](https://docs.microsoft.com/rest/api/automation/softwareupdateconfigurations/create)。 此外，还有一个示例 Runbook，可用于创建每周更新部署。 若要了解有关此 Runbook 的详细信息，请参阅[为资源组中的一个或多个 VM 创建每周更新部署](https://gallery.technet.microsoft.com/scriptcenter/Create-a-weekly-update-2ad359a1)。

### <a name="view-results-of-an-update-deployment"></a>查看更新部署结果

在计划性部署开始后，可以在“更新管理”窗口的“更新部署”选项卡上查看该部署的状态。

如果部署当前正在运行，则其状态为“正在进行。” 成功完成后，状态将更改为“已成功。” 如果部署中任何更新失败，则状态为“部分失败。”

选择已完成的更新部署，查看该部署的仪表板。

:::image type="content" source="./media/tutorial-monitoring/manageupdates-view-results.png" alt-text="启用更新管理解决方案":::

“更新结果”磁贴显示了 VM 上更新和部署结果的总数的概要。 右侧的表显示了每个更新的细目以及安装结果。 每个结果都具有下列值之一：

* **未尝试** ：未安装更新。 基于定义的维护时段持续时间，没有足够的可用时间。
* **成功** ：更新成功。
* **失败** ：更新失败。

若要查看部署创建的所有日志条目，请选择“所有日志”。

选择“输出”磁贴，查看负责管理目标 VM 更新部署的 runbook 的作业流。

若要查看有关任何部署错误的详细信息，请选择“错误”。

## <a name="monitor-changes-and-inventory"></a>监视器更改和清单

可以收集和查看清单，了解计算机上的软件、文件、Linux 守护程序、Windows 服务和 Windows 注册表项。 跟踪计算机的配置有助于查明环境中的操作问题，更好地了解计算机的状态。

<!--Not Available on ### Enable change and inventory management-->
<!--Not Available on ### Track changes-->
<!--Not Available on ### View inventory-->
<!--Not Available on ### Monitor activity logs and changes-->

## <a name="next-steps"></a>后续步骤

在本教程中，你配置并查看了 VM 的更改跟踪和更新管理。 你已了解如何：

> [!div class="checklist"]
> * 创建资源组和 VM。
> * 管理 Windows 更新。
> * 监视器更改和清单。

请转到下一教程，了解如何监视 VM。

> [!div class="nextstepaction"]
> [监视虚拟机](tutorial-monitor.md)

<!-- Update_Description: update meta properties, wording update, update link -->