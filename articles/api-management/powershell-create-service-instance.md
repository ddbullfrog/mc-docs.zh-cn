---
title: 快速入门 - 使用 PowerShell 创建 Azure API 管理实例
description: 使用 Azure PowerShell 新建 Azure API 管理实例。
services: api-management
documentationcenter: ''
author: Johnnytechn
ms.service: api-management
ms.topic: quickstart
ms.custom: mvc
origin.date: 11/15/2017
ms.date: 09/29/2020
ms.author: v-johya
ms.openlocfilehash: 81a0c593ce3c905aa98407e04a7333553eccdffc
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871301"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-by-using-powershell"></a>快速入门：使用 PowerShell 创建新的 Azure API 管理服务实例

Azure API 管理 (APIM) 可帮助组织将 API 发布给外部、合作伙伴和内部开发人员，以充分发挥其数据和服务的潜力。 API 管理通过开发人员参与、商业洞察力、分析、安全性和保护提供了核心竞争力以确保成功的 API 程序。 使用 APIM 可以为在任何位置托管的现有后端服务创建和管理新式 API 网关。 有关详细信息，请参阅[概述](api-management-key-concepts.md)。

本快速入门介绍使用 Azure PowerShell cmdlet 新建 API 管理实例的步骤。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

如果选择在本地安装并使用 PowerShell，则本教程需要安装 Azure PowerShell 模块 1.0 或更高版本。 运行 `Get-Module -ListAvailable Az` 即可查找版本。 如果需要进行升级，请参阅 [Install Azure PowerShell module](https://docs.microsoft.com/powershell/azure/install-Az-ps)（安装 Azure PowerShell 模块）。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 以创建与 Azure 的连接。

## <a name="create-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

以下命令在“中国北部”位置创建名为“myResourceGroup”的资源组：

```azurepowershell
New-AzResourceGroup -Name myResourceGroup -Location ChinaNorth
```

## <a name="create-an-api-management-service"></a>创建 API 管理服务

现在，你已有了一个资源组，可以创建 API 管理服务实例了。 使用 [New-AzApiManagement](https://docs.microsoft.com/powershell/module/az.apimanagement/new-azapimanagement) 创建一个 API 管理服务实例，并提供服务名称和发布者详细信息。 服务名称在 Azure 中必须独一无二。

在下面的示例中，使用“myapim”作为服务名称。 将该名称更新为唯一值。 同时更新 API 发布者的组织名称和管理员电子邮件地址以接收通知。

默认情况下，该命令在“开发人员”层创建实例，这是评估 Azure API 管理的一个经济选择。 此层不用于生产。 有关对 API 管理层进行缩放的详细信息，请参阅[升级和缩放](upgrade-and-scale.md)。

> [!NOTE]
> 此操作将运行较长时间。 在此层中创建和激活 API 管理服务可能需要 30 到 40 分钟。

```azurepowershell
New-AzApiManagement -Name "myapim" -ResourceGroupName "myResourceGroup" `
  -Location "China North" -Organization "Contoso" -AdminEmail "admin@contoso.com" 
```

当该命令返回时，运行 [Get-AzApiManagement](https://docs.microsoft.com/powershell/module/az.apimanagement/get-azapimanagement) 可查看 Azure API 管理服务的属性。 激活后，预配状态为“Succeeded”，并且服务实例具有多个关联的 URL。 例如： 。

```azurepowershell
Get-AzApiManagement -Name "myapim" -ResourceGroupName "myResourceGroup" 
```

示例输出：

```console
PublicIPAddresses                     : {203.0.113.1}
PrivateIPAddresses                    :
Id                                    : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.ApiManagement/service/myapim
Name                                  : myapim
Location                              : China North
Sku                                   : Developer
Capacity                              : 1
CreatedTimeUtc                        : 9/9/2020 9:07:43 PM
ProvisioningState                     : Succeeded
RuntimeUrl                            : https://myapim.azure-api.net
RuntimeRegionalUrl                    : https://myapi-chinanorth-01.regional.azure-api.net
PortalUrl                             : https://myapim.portal.azure-api.net
DeveloperPortalUrl                    : https://myapim.developer.azure-api.net
ManagementApiUrl                      : https://myapim.management.azure-api.net
ScmUrl                                : https://myapim.scm.azure-api.net
PublisherEmail                        : admin@contoso.com
OrganizationName                      : Contoso
NotificationSenderEmail               : apimgmt-noreply@mail.windowsazure.cn
VirtualNetwork                        :
VpnType                               : None
PortalCustomHostnameConfiguration     :
ProxyCustomHostnameConfiguration      : {myapim.azure-api.net}
ManagementCustomHostnameConfiguration :
ScmCustomHostnameConfiguration        :
DeveloperPortalHostnameConfiguration  :
SystemCertificates                    :
Tags                                  : {}
AdditionalRegions                     : {}
SslSetting                            : Microsoft.Azure.Commands.ApiManagement.Models.PsApiManagementSslSetting
Identity                              :
EnableClientCertificate               :
ResourceGroupName                     : myResourceGroup

```

部署 API 管理服务实例后，便可以使用它了。 从[导入并发布第一个 API](import-and-publish.md) 教程开始。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组和所有相关资源，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 命令将其删除。

```azurepowershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [导入和发布第一个 API](import-and-publish.md)

