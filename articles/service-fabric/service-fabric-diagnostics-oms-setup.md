---
title: 使用 Azure Monitor 日志设置监视
description: 了解如何设置 Azure Monitor 来可视化和分析事件，以监视 Azure Service Fabric 群集。
ms.topic: conceptual
origin.date: 02/20/2019
author: rockboyfor
ms.date: 09/14/2020
ms.testscope: yes|no
ms.testdate: 09/14/2020null
ms.author: v-yeche
ms.openlocfilehash: dc84747733940f8286214bc848f56e96cda80fcf
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211434"
---
<!--Verified successfully on 09/07/2020-->
# <a name="set-up-azure-monitor-logs-for-a-cluster"></a>为群集设置 Azure Monitor 日志

要监视群集级别的事件，建议使用 Azure Monitor 日志。 可以通过 Azure 资源管理器、PowerShell 或 Azure 市场设置 Log Analytics 工作区。 对于你的部署，如果你维护其更新后的资源管理器模板供将来使用，请使用同一模板来设置 Azure Monitor 日志环境。 如果部署的群集已启用诊断，则通过市场部署更方便。 如果你在要部署到的帐户中没有订阅级别访问权限，请通过 PowerShell 或资源管理器模板进行部署。

> [!NOTE]
> 若要设置 Azure Monitor 日志来监视群集，需要启用诊断来查看群集级别事件或平台级别事件。 有关详细信息，请参阅[如何在 Windows 群集中设置诊断](service-fabric-diagnostics-event-aggregation-wad.md)和[如何在 Linux 群集中设置诊断](service-fabric-diagnostics-oms-syslog.md)

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

<!--Not Available on ## Deploy a Log Analytics workspace by using Azure Marketplace-->
<!--Not Available on Service Fabric Analytics-->

## <a name="deploy-azure-monitor-logs-with-azure-resource-manager"></a>使用 Azure 资源管理器部署 Azure Monitor 日志

使用资源管理器模板部署群集时，该模板还会新建一个 Log Analytics 工作区，向其添加 Service Fabric 解决方案，并将其配置为读取相应存储表中的数据。

可使用[此示例模板](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure)，还可进行修改，使其满足你的要求。 此模板执行以下操作

* 创建一个 5 节点 Service Fabric 群集
* 创建 Log Analytics 工作区和 Service Fabric 解决方案
* 配置 Log Analytics 代理以收集 2 个示例性能计数器并将其发送到工作区
* 配置 WAD 以收集 Service Fabric 并将其发送到 Azure 存储表 (WADServiceFabric*EventTable)
* 配置 Log Analytics 工作区以从这些表中读取事件

<!--MOONCAKE CUSTOMIZATION-->

>  [!NOTE]
> 为了适合 Azure 中国云环境，必须修改从 GitHub 存储库“Azure-Sample”下载或引用的模板。
> 例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“cloudapp.chinacloudapi.cn”）；必要时更改某些不受支持的位置、VM 映像、VM 大小、SKU 以及资源提供程序的 API 版本。
>
> 在本文中，请在运行 powershell 命令之前更改以下项。
> 1. 更改 `computeLocation` 和 `omsRegion` 参数的以下属性。
>       * 将 `defaultValue` 的属性更新为 `chinaeast2`。
>       * 将 `allowedValues` 的属性更新为 `["chinaeast","chinaeast2","chinanorth","chinanorth2"]`
> 2. 将所有终结点后缀从 `windows.net` 更改为 `chinacloudapi.cn`。

<!--MOONCAKE CUSTOMIZATION-->

可以通过使用 Azure PowerShell 模块中的 `New-AzResourceGroupDeployment` API，将该模板作为资源管理器升级部署到群集。 示例命令如下：

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resourceGroupName>" -TemplateFile "<templatefile>.json" 
``` 

Azure 资源管理器可检测出此命令要更新到现有资源。 它仅处理驱动现有部署的模板和提供的新模板之间的变化。

## <a name="deploy-azure-monitor-logs-with-azure-powershell"></a>使用 Azure PowerShell 部署 Azure Monitor 日志

也可以使用 `New-AzOperationalInsightsWorkspace` 命令，通过 PowerShell 来部署日志分析资源。 要使用此方法，请确保已安装 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps)。 使用此脚本新建 Log Analytics 工作区，并向其添加 Service Fabric 解决方案： 

```powershell

$SubID = "<subscription ID>"
$ResourceGroup = "<Resource group name>"
$Location = "<Resource group location>"
$WorkspaceName = "<Log Analytics workspace name>"
$solution = "ServiceFabric"

# Sign in to Azure and access the correct subscription
Connect-AzAccount -Environment AzureChinaCloud
Select-AzSubscription -SubscriptionId $SubID 

# Create the resource group if needed
try {
    Get-AzResourceGroup -Name $ResourceGroup -ErrorAction Stop
} catch {
    New-AzResourceGroup -Name $ResourceGroup -Location $Location
}

New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku Standard -ResourceGroupName $ResourceGroup
Set-AzOperationalInsightsIntelligencePack -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -IntelligencePackName $solution -Enabled $true

```

完成后，请按照前一部分中的步骤将 Azure Monitor 日志连接到相应的存储帐户。

还可添加其他解决方案或使用 PowerShell 对 Log Analytics 工作区进行其他修改。 有关详细信息，请参阅[使用 PowerShell 管理 Azure Monitor 日志](../azure-monitor/platform/powershell-workspace-configuration.md)。

## <a name="next-steps"></a>后续步骤
* [将 Log Analytics 代理部署到节点上](service-fabric-diagnostics-oms-agent.md)，以收集性能计数器、docker 统计信息和容器日志
* 掌握 Azure Monitor 日志中提供的[日志搜索和查询](../azure-monitor/log-query/log-query-overview.md)功能
* [使用视图设计器在 Azure Monitor 日志中创建自定义视图](../azure-monitor/platform/view-designer.md)

<!-- Update_Description: new article about service fabric diagnostics oms setup -->
<!--NEW.date: 09/14/2020-->