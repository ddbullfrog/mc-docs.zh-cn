---
title: PowerShell：使用流量管理器进行缩放
description: 了解如何使用 Azure PowerShell 自动部署和管理应用服务。 本示例演示如何使用流量管理器在全球范围内缩放应用。
tags: azure-service-management
ms.assetid: 470f0129-1efe-462c-a029-5c66e04158a8
ms.topic: sample
origin.date: 03/20/2017
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: a4b4552b02c65703335e2b17452888853a006e34
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170457"
---
# <a name="scale-a-web-app-worldwide-with-a-high-availability-architecture"></a>缩放具有高可用性体系结构的全球 Web 应用

在此方案中，将创建一个资源组、两个应用服务计划、两个 Web 应用、一个流量管理器配置文件和两个流量管理器终结点。 完成本练习后，会获得一个高可用性体系结构，它基于最低网络延迟提供 Web 应用的全局可用性。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/powershell/azure/)中的说明安装 Azure PowerShell，并运行 `Connect-AzAccount -Environment AzureChinaCloud` 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```powershell
# Generates a Random Value
$Random=(New-Guid).ToString().Substring(0,8)

# Variables
$ResourceGroupName="myResourceGroup$Random"
$App1Name="AppServiceTM1$Random"
$App2Name="AppServiceTM2$Random"
$Location1="ChinaEast"
$Location2="ChinaNorth"

# Create a Resource Group
New-AzResourceGroup -Name $ResourceGroupName -Location $Location1

# Create Traffic Manager Profile
New-AzTrafficManagerProfile -Name "$ResourceGroupName-tmp" -ResourceGroupName $ResourceGroupName -TrafficRoutingMethod Performance -MonitorPath '/' -MonitorProtocol "HTTP" -RelativeDnsName $ResourceGroupName -Ttl 30 -MonitorPort 80

# Create an App Service Plan
New-AzAppservicePlan -Name "$App1Name-Plan" -ResourceGroupName $ResourceGroupName -Location $Location1 -Tier Standard
New-AzAppservicePlan -Name "$App2Name-Plan" -ResourceGroupName $ResourceGroupName -Location $Location2 -Tier Standard

# Create a Web App in the App Service Plan
$App1ResourceId=(New-AzWebApp -Name $App1Name -ResourceGroupName $ResourceGroupName -Location $Location1 -AppServicePlan "$App1Name-Plan").Id
$App2ResourceId=(New-AzWebApp -Name $App2Name -ResourceGroupName $ResourceGroupName -Location $Location2 -AppServicePlan "$App2Name-Plan").Id

# Create Traffic Manager Endpoints for Web Apps
New-AzTrafficManagerEndpoint -Name "$App1Name-$Location1" -ResourceGroupName $ResourceGroupName -ProfileName "$ResourceGroupName-tmp" -Type AzureEndpoints -TargetResourceId $App1ResourceId -EndpointStatus "Enabled"
New-AzTrafficManagerEndpoint -Name "$App2Name-$Location2" -ResourceGroupName $ResourceGroupName -ProfileName "$ResourceGroupName-tmp" -Type AzureEndpoints -TargetResourceId $App2ResourceId -EndpointStatus "Enabled"
```
## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile) | 创建流量管理器配置文件。 |
| [New-AzAppServicePlan](https://docs.microsoft.com/powershell/module/az.websites/new-azappserviceplan) | 创建应用服务计划。 |
| [New-AzWebApp](https://docs.microsoft.com/powershell/module/az.websites/new-azwebapp) | 创建 Web 应用。 |
| [New-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) | 在流量管理器配置文件中创建终结点。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/)。

可以在 [Azure PowerShell 示例](../samples-powershell.md)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。
