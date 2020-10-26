---
title: 从 Powershell 中的群集删除应用程序
description: Azure PowerShell 脚本示例 - 从 Service Fabric 群集中删除应用程序。
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
ms.openlocfilehash: f28450be7e97ef7685dacc28f107fa49b0c88225
ms.sourcegitcommit: 221c32fe6f618679a63f148da7382bc9e495f747
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211855"
---
# <a name="remove-an-application-from-a-service-fabric-cluster-using-powershell"></a>使用 Powershell 从 Service Fabric 群集中删除应用程序

此示例脚本删除了正在运行的 Service Fabric 应用程序实例，并从群集中取消注册了应用程序类型和版本。  删除应用程序实例时也会删除与该应用程序关联的所有正在运行的服务实例。 根据需要自定义参数。 

必要时，使用 [Service Fabric SDK](../service-fabric-get-started.md) 安装 Service Fabric PowerShell 模块。 

## <a name="sample-script"></a>示例脚本

```powershell
# Variables
$endpoint = 'mysftestcluster.chinaeast.cloudapp.chinacloudapi.cn:19000'
$thumbprint = '2779F0BB9A969FB88E04915FFE7955D0389DA7AF'
$packagepath="C:\Users\sfuser\Documents\Visual Studio 2017\Projects\MyApplication\MyApplication\pkg\Release"

# Connect to the cluster using a client certificate.
Connect-ServiceFabricCluster -ConnectionEndpoint $endpoint `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint $thumbprint `
          -FindType FindByThumbprint -FindValue $thumbprint `
          -StoreLocation CurrentUser -StoreName My

# Remove an application instance
Remove-ServiceFabricApplication -ApplicationName fabric:/MyApplication

# Unregister the application type
Unregister-ServiceFabricApplicationType -ApplicationTypeName MyApplicationType -ApplicationTypeVersion 1.0.0

```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| Command | 说明 |
|---|---|
| [Remove-ServiceFabricApplication](https://docs.microsoft.com/powershell/module/servicefabric/remove-servicefabricapplication?view=azureservicefabricps) | 从群集中删除正在运行的 Service Fabric 应用程序实例。  |
| [Unregister-ServiceFabricApplicationType](https://docs.microsoft.com/powershell/module/servicefabric/unregister-servicefabricapplicationtype?view=azureservicefabricps) | 从群集中注销 Service Fabric 应用程序类型和版本。 |

## <a name="next-steps"></a>后续步骤

有关 Service Fabric PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/azure/service-fabric/overview?view=azureservicefabricps)。

可以在 [Azure PowerShell 示例](../service-fabric-powershell-samples.md)中找到 Azure Service Fabric 的其他 Powershell 示例。

<!-- Update_Description: update meta properties, wording update, update link -->