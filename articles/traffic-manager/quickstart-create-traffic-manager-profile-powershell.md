---
title: 快速入门 - 创建配置文件以实现应用程序的高可用性 - Azure PowerShell - Azure 流量管理器
description: 本快速入门文章介绍如何创建流量管理器配置文件，以生成高度可用的 Web 应用程序。
services: traffic-manager
mnager: twooley
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.service: traffic-manager
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 08/25/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: yes
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.openlocfilehash: 2376164639c3c66edbeccfc0b53d7a4150e0d455
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395427"
---
# <a name="quickstart-create-a-traffic-manager-profile-for-a-highly-available-web-application-using-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建流量管理器配置文件以实现 Web 应用程序的高可用性

本快速入门介绍如何创建流量管理器配置文件，以便实现 Web 应用程序的高度可用性。

在本快速入门中，我们将创建 Web 应用程序的两个实例。 每个实例在不同的 Azure 区域运行。 需根据[终结点优先级](traffic-manager-routing-methods.md#priority-traffic-routing-method)创建流量管理器配置文件。 此配置文件将用户流量定向到运行 Web 应用程序的主站点。 流量管理器持续监视 Web 应用程序。 如果主站点不可用，它会提供目标为备份站点的自动故障转移。

如果还没有 Azure 订阅，请现在就创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

<!--[!INCLUDE [cloud-shell-powershell](../../../includes/cloud-shell-powershell.md)]-->

如果选择在本地安装并使用 PowerShell，则本文需要 Azure PowerShell 模块 5.4.1 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 来创建与 Azure 的连接。

## <a name="create-a-resource-group"></a>创建资源组。
使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建资源组。

```powershell

# Variables
$Location1="ChinaNorth"

# Create a Resource Group
New-AzResourceGroup -Name MyResourceGroup -Location $Location1
```

## <a name="create-a-traffic-manager-profile"></a>创建流量管理器配置文件

使用 [New-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile) 创建流量管理器配置文件，以根据终结点优先级定向用户流量。

```powershell

# Generates a random value
$Random=(New-Guid).ToString().Substring(0,8)
$mytrafficmanagerprofile="mytrafficmanagerprofile$Random"

New-AzTrafficManagerProfile `
-Name $mytrafficmanagerprofile `
-ResourceGroupName MyResourceGroup `
-TrafficRoutingMethod Priority `
-MonitorPath '/' `
-MonitorProtocol "HTTP" `
-RelativeDnsName $mytrafficmanagerprofile `
-Ttl 30 `
-MonitorPort 80
```

## <a name="create-web-apps"></a>创建 Web 应用

本快速入门需要两个部署在两个不同的 Azure 区域（中国北部和中国东部）的 Web 应用程序实例。**** 每个都可以充当流量管理器的主终结点和故障转移终结点。

### <a name="create-web-app-service-plans"></a>创建 Web 应用服务计划
使用 [New-AzAppServicePlan](https://docs.microsoft.com/powershell/module/az.websites/new-azappserviceplan) 为要部署在两个不同 Azure 区域中的两个 Web 应用程序实例创建 Web 应用服务计划。

```powershell

# Variables
$App1Name="AppServiceTM1$Random"
$App2Name="AppServiceTM2$Random"
$Location1="ChinaNorth"
$Location2="ChinaEast"

# Create an App service plan
New-AzAppservicePlan -Name "$App1Name-Plan" -ResourceGroupName MyResourceGroup -Location $Location1 -Tier Standard
New-AzAppservicePlan -Name "$App2Name-Plan" -ResourceGroupName MyResourceGroup -Location $Location2 -Tier Standard

```
### <a name="create-a-web-app-in-the-app-service-plan"></a>在应用服务计划中创建 Web 应用
在应用服务计划中使用 [New-AzWebApp](https://docs.microsoft.com/powershell/module/az.websites/new-azwebapp) 在“中国北部”和“中国东部”Azure 区域中创建 Web 应用程序的两个实例。****

```powershell
$App1ResourceId=(New-AzWebApp -Name $App1Name -ResourceGroupName MyResourceGroup -Location $Location1 -AppServicePlan "$App1Name-Plan").Id
$App2ResourceId=(New-AzWebApp -Name $App2Name -ResourceGroupName MyResourceGroup -Location $Location2 -AppServicePlan "$App2Name-Plan").Id

```

## <a name="add-traffic-manager-endpoints"></a>添加流量管理器终结点
使用 [New-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) 将两个 Web 应用作为流量管理器终结点添加到流量管理器配置文件，如下所示：
- 将“中国北部”Azure 区域中的 Web 应用添加为主要终结点，以路由所有用户流量。** 
- 将“中国东部”Azure 区域中的 Web 应用添加为故障转移终结点。** 当主终结点不可用时，流量自动路由到故障转移终结点。

```powershell
New-AzTrafficManagerEndpoint -Name "$App1Name-$Location1" `
-ResourceGroupName MyResourceGroup `
-ProfileName "$mytrafficmanagerprofile" `
-Type AzureEndpoints `
-TargetResourceId $App1ResourceId `
-EndpointStatus "Enabled"

New-AzTrafficManagerEndpoint -Name "$App2Name-$Location2" `
-ResourceGroupName MyResourceGroup `
-ProfileName "$mytrafficmanagerprofile" `
-Type AzureEndpoints `
-TargetResourceId $App2ResourceId `
-EndpointStatus "Enabled"
```

## <a name="test-traffic-manager-profile"></a>测试流量管理器配置文件

在此部分，需检查流量管理器配置文件的域名。 此外还需将主终结点配置为不可用。 最后可以看到该 Web 应用仍然可用。 这是因为流量管理器将流量发送到故障转移终结点。

### <a name="determine-the-dns-name"></a>确定 DNS 名称

使用 [Get-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.trafficmanager/get-aztrafficmanagerprofile) 确定流量管理器配置文件的 DNS 名称。

```powershell
Get-AzTrafficManagerProfile -Name $mytrafficmanagerprofile `
-ResourceGroupName MyResourceGroup
```

复制 **RelativeDnsName** 值。 流量管理器配置文件的 DNS 名称为“*http://<* relativednsname *>.trafficmanager.cn”* 。 

### <a name="view-traffic-manager-in-action"></a>查看正在运行的流量管理器
1. 在 Web 浏览器中输入流量管理器配置文件的 DNS 名称 (*http://<* relativednsname *>.trafficmanager.cn*)，以查看 Web 应用的默认网站。

    > [!NOTE]
    > 在本快速入门方案中，所有请求都路由到主终结点。 它设置为“优先级 1”。 
2. 若要查看流量管理器故障转移如何进行，请使用 [Disable-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/disable-aztrafficmanagerendpoint) 禁用主要站点。

    ```powershell
    Disable-AzTrafficManagerEndpoint -Name $App1Name-$Location1 `
    -Type AzureEndpoints `
    -ProfileName $mytrafficmanagerprofile `
    -ResourceGroupName MyResourceGroup `
    -Force
    ```
3. 复制流量管理器配置文件的 DNS 名称 (http://<relativednsname>.trafficmanager.cn)，以在新的 Web 浏览器会话中查看该网站。 
4. 验证 Web 应用是否仍然可用。

## <a name="clean-up-resources"></a>清理资源

完成后，请使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 删除资源组、Web 应用程序和所有相关资源。

```powershell
Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，我们创建了一个可为 Web 应用程序提供高可用性的流量管理器配置文件。 若要详细了解如何路由流量，请继续学习流量管理器教程。

> [!div class="nextstepaction"]
> [流量管理器教程](tutorial-traffic-manager-improve-website-response.md)

<!-- Update_Description: update meta properties, wording update, update link -->