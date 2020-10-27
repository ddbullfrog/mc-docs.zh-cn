---
title: 在 Powershell 中升级 Service Fabric 应用程序
description: Azure PowerShell 脚本示例 - 使用 Powershell 升级和监控 Azure Service Fabric 应用程序。
services: service-fabric
manager: chackdan
tags: azure-service-management
ms.service: service-fabric
ms.workload: multiple
ms.topic: sample
origin.date: 01/18/2018
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: 9ed4f9952c78545e5425961483c8209e897dc577
ms.sourcegitcommit: 221c32fe6f618679a63f148da7382bc9e495f747
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211885"
---
# <a name="upgrade-a-service-fabric-application"></a>升级 Service Fabric 应用程序

此示例脚本可将正在运行的 Service Fabric 应用程序实例升级为版本 1.3.0。 该脚本会将新应用程序包复制到群集映像存储区，注册应用程序类型并删除不必要的应用程序包。  该脚本会启动受监控的升级，并持续检查升级状态，直到升级完成或回滚。 根据需要自定义参数。 

必要时，使用 [Service Fabric SDK](../service-fabric-get-started.md) 安装 Service Fabric PowerShell 模块。 

## <a name="sample-script"></a>示例脚本

```powershell
## Variables
$ApplicationPackagePath = "C:\Users\sfuser\documents\visual studio 2017\Projects\Voting\Voting\pkg\Debug"
$ApplicationName = "fabric:/Voting"
$ApplicationTypeName = "VotingType"
$ApplicationTypeVersion = "1.3.0"
$imageStoreConnectionString = "fabric:ImageStore"
$CopyPackageTimeoutSec = 600
$CompressPackage = $false

## Check existence of the application
$oldApplication = Get-ServiceFabricApplication -ApplicationName $ApplicationName

if (!$oldApplication)
{
    $errMsg = "Application '$ApplicationName' doesn't exist in cluster."
    throw $errMsg
}
else
{
    ## Check upgrade status
    $upgradeStatus = Get-ServiceFabricApplicationUpgrade -ApplicationName $ApplicationName
    if ($upgradeStatus.UpgradeState -ne "RollingBackCompleted" -and $upgradeStatus.UpgradeState -ne "RollingForwardCompleted" -and $upgradeStatus.UpgradeState -ne "Failed")
    {
        $errMsg = "An upgrade for the application '$ApplicationTypeName' is already in progress."
        throw $errMsg
    }

    $reg = Get-ServiceFabricApplicationType -ApplicationTypeName $ApplicationTypeName | Where-Object  { $_.ApplicationTypeVersion -eq $ApplicationTypeVersion }
    if ($reg)
    {
        Write-Host 'Application Type '$ApplicationTypeName' and Version '$ApplicationTypeVersion' was already registered with Cluster, unregistering it...'
        $reg | Unregister-ServiceFabricApplicationType -Force
    }

    ## Copy application package to image store
    $applicationPackagePathInImageStore = $ApplicationTypeName
    Write-Host "Copying application package to image store..."
    Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $ApplicationPackagePath -ImageStoreConnectionString $imageStoreConnectionString -ApplicationPackagePathInImageStore $applicationPackagePathInImageStore -TimeOutSec $CopyPackageTimeoutSec -CompressPackage:$CompressPackage 
    if(!$?)
    {
        throw "Copying of application package to image store failed. Cannot continue with registering the application."
    }

    ## Register application type
    Write-Host "Registering application type..."
    Register-ServiceFabricApplicationType -ApplicationPathInImageStore $applicationPackagePathInImageStore
    if(!$?)
    {
        throw "Registration of application type failed."
    }

    # Remove the application package to free system resources.
    Remove-ServiceFabricApplicationPackage -ImageStoreConnectionString $imageStoreConnectionString -ApplicationPackagePathInImageStore $applicationPackagePathInImageStore
    if(!$?)
    {
        Write-Host "Removing the application package failed."
    }

    ## Start monitored application upgrade
    try
    {
        Write-Host "Start upgrading application..." 
        Start-ServiceFabricApplicationUpgrade -ApplicationName $ApplicationName -ApplicationTypeVersion $ApplicationTypeVersion -HealthCheckStableDurationSec 60 -UpgradeDomainTimeoutSec 1200 -UpgradeTimeout 3000 -FailureAction Rollback -Monitored
    }
    catch
    {
        Write-Host ("Error starting upgrade. " + $_)

        Write-Host "Unregister application type '$ApplicationTypeName' and version '$ApplicationTypeVersion' ..."
        Unregister-ServiceFabricApplicationType -ApplicationTypeName $ApplicationTypeName -ApplicationTypeVersion $ApplicationTypeVersion -Force
        throw
    }

    do
    {
        Write-Host "Waiting for upgrade..."
        Start-Sleep -Seconds 3
        $upgradeStatus = Get-ServiceFabricApplicationUpgrade -ApplicationName $ApplicationName
    } while ($upgradeStatus.UpgradeState -ne "RollingBackCompleted" -and $upgradeStatus.UpgradeState -ne "RollingForwardCompleted" -and $upgradeStatus.UpgradeState -ne "Failed")

    if($upgradeStatus.UpgradeState -eq "RollingForwardCompleted")
    {
        Write-Host "Upgrade completed successfully."
    }
    elseif($upgradeStatus.UpgradeState -eq "RollingBackCompleted")
    {
        Write-Error "Upgrade was Rolled back."
    }
    elseif($upgradeStatus.UpgradeState -eq "Failed")
    {
        Write-Error "Upgrade Failed."
    }
}
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [Get-ServiceFabricApplication](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricapplication?view=azureservicefabricps) | 获取 Service Fabric 群集中的所有应用程序或特定应用程序。  |
| [Get-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricapplicationupgrade?view=azureservicefabricps) | 获取 Service Fabric 应用程序的升级状态。 |
| [Get-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricapplicationtype?view=azureservicefabricps) | 获取在 Service Fabric 群集上注册的 Service Fabric 应用程序类型。 |
| [Unregister-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/module/servicefabric/unregister-servicefabricapplicationtype?view=azureservicefabricps) | 注销 Service Fabric 应用程序类型。  |
| [Copy-ServiceFabricApplicationPackage](https://docs.microsoft.com/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) | 将 Service Fabric 应用程序包复制到映像存储。  |
| [Register-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) | 注册 Service Fabric 应用程序类型。 |
| [Start-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricapplicationupgrade?view=azureservicefabricps) | 将 Service Fabric 应用程序升级为指定的应用程序类型版本。 |
| [Remove-ServiceFabricApplicationPackage](https://docs.microsoft.com/powershell/module/servicefabric/remove-servicefabricapplicationpackage?view=azureservicefabricps) | 从映像存储区中删除 Service Fabric 应用程序包。|

## <a name="next-steps"></a>后续步骤

有关 Service Fabric PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/service-fabric/overview?view=azureservicefabricps)。

可以在 [Azure PowerShell 示例](../service-fabric-powershell-samples.md)中找到 Azure Service Fabric 的其他 Powershell 示例。

<!-- Update_Description: update meta properties, wording update, update link -->