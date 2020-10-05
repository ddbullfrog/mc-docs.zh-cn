---
title: 安装 Azure Analysis Services 的本地数据网关 | Azure
description: 了解如何安装和配置本地数据网关，以从 Azure Analysis Services 服务器连接到本地数据源。
ms.service: azure-analysis-services
ms.topic: conceptual
origin.date: 07/29/2020
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.reviewer: minewiskan
ms.openlocfilehash: 547944e1044b63ff3e14f67765f7d99bfa6b4e2b
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146754"
---
# <a name="install-and-configure-an-on-premises-data-gateway"></a>安装并配置本地数据网关

当同一区域中的一个或多个 Azure Analysis Services 服务器连接到本地数据源时，需要本地数据网关。  虽然你安装的网关与其他服务（例如 Power BI、Power Apps 和逻辑应用）使用的网关相同，但在针对 Azure Analysis Services 进行安装时，有一些需要完成的额外步骤。 本安装文章专门针对 **Azure Analysis Services**。 

若要详细了解 Azure Analysis Services 如何使用网关，请参阅[连接到本地数据源](analysis-services-gateway.md)。 若要总体了解有关高级安装方案和网关的更多信息，请参阅[本地数据网关文档](https://docs.microsoft.com/data-integration/gateway/service-gateway-onprem)。

## <a name="prerequisites"></a>先决条件

**最低要求：**

* .NET 4.5 Framework
* 64 位版本的 Windows 8/Windows Server 2012 R2（或更高版本）

**推荐：**

* 8 核 CPU
* 8 GB 内存
* 64 位版本的 Windows 8/Windows Server 2012 R2（或更高版本）

**重要注意事项：**

* 在安装过程中将网关注册到 Azure 时，会选择订阅的默认区域。 可以选择不同的订阅和区域。 如果在多个区域中具有服务器，则必须为每个区域安装一个网关。 
* 不能在域控制器上安装网关。
* 一台计算机上只能安装一个网关。
* 在计算机处于开启但未处于休眠状态下安装网关。
* 请勿在仅通过无线方式连接到网络的计算机上安装网关。 否则，可能会降低性能。
* 安装网关时，你用来登录到计算机的用户帐户必须具有“作为服务登录”权限。 安装完成后，本地数据网关服务使用 NT SERVICE\PBIEgwService 帐户作为服务登录。 可以在安装期间指定一个不同的帐户，也可以在安装完成后在“服务”中指定一个不同的帐户。 请确保组策略设置同时允许你在安装时登录的帐户以及你选择的具有“作为服务登录”权限的服务帐户。
* 在 Azure AD 中使用与要在其中注册网关的订阅相同[租户](https://docs.microsoft.com/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant)的帐户登录到 Azure。 安装和注册网关时不支持 Azure B2B（来宾）帐户。
    
    <!--MOONCAKE: Available on [tenant](https://docs.microsoft.com/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant)-->
    
* 如果数据源位于 Azure 虚拟网络 (VNet) 上，则必须配置 [AlwaysUseGateway](analysis-services-vnet-gateway.md) 服务器属性。

<a name="download"></a>
## <a name="download"></a>下载

 [下载网关](https://go.microsoft.com/fwlink/?LinkId=820925&clcid=0x409)

<a name="install"></a>
## <a name="install"></a>安装

1. 运行安装程序。

2. 选择“本地数据网关”。 

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-installer-select.png" alt-text="Select":::

2. 选择位置，接受条款，并单击“安装”。 

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-installer-accept.png" alt-text="Select":::

3. 登录 Azure。 该帐户必须在租户的 Azure Active Directory 中。 这是网关管理员使用的帐户。 安装和注册网关时不支持 Azure B2B（来宾）帐户。

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-installer-account.png" alt-text="Select":::

    > [!NOTE]
    > 如果使用域帐户登录，它将映射到你在 Azure AD 中的组织帐户。 你的组织帐户将用作网关管理员。

<a name="register"></a>
## <a name="register"></a>注册

若要在 Azure 中创建网关资源，必须将安装的本地实例注册到网关云服务。 

1. 选择“在此计算机上注册新网关”  。

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-register-new.png" alt-text="Select":::

2. 键入网关的名称和恢复密钥。 默认情况下，网关使用订阅的默认区域。 如需选择不同的区域，请选择“更改区域”。 

    > [!IMPORTANT]
    > 将恢复密钥保存在安全位置。 接管、迁移或还原网关时需要使用恢复密钥。 

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-register-name.png" alt-text="Select":::

<a name="create-resource"></a>
## <a name="create-an-azure-gateway-resource"></a>创建 Azure 网关资源

安装并注册网关后，需要在 Azure 中创建网关资源。 使用注册网关时所用的同一帐户登录到 Azure。

1. 在 Azure 门户中单击“创建资源”，接着搜索“本地数据网关”，然后单击“创建”。   

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-new-azure-resource.png" alt-text="Select":::

2. 在“创建连接网关”中，输入以下设置： 

    * **名称**：输入网关资源的名称。 

    * 订阅：选择要与网关资源关联的 Azure 订阅。 

        默认订阅取决于用来登录的 Azure 帐户。

    * **资源组**：创建资源组，或选择现有资源组。

    * **位置**：选择网关的注册区域。

    * **安装名称**：如果尚未选择网关安装，请选择在计算机上安装并注册的网关。 

    完成后，单击“创建”  。

<a name="connect-servers"></a>
## <a name="connect-gateway-resource-to-server"></a><a name="connect-gateway-resource-to-server"></a>将网关资源连接到服务器

> [!NOTE]
> 门户不支持从服务器连接到不同订阅中的网关资源，但 PowerShell 支持。

# <a name="portal"></a>[门户](#tab/azure-portal)

1. 在 Azure Analysis Services 服务器概述中，单击“本地数据网关”****。

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-connect-server.png" alt-text="Select":::

2. 在“选取要连接的本地数据网关”****，选择你的网关资源，然后单击“连接所选网关”****。

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-connect-resource.png" alt-text="Select":::

    > [!NOTE]
    > 如果列表中不显示你的网关，很可能是你的服务器与你注册网关时指定的区域不在同一个区域。

    在服务器和网关资源之间成功建立连接以后，状态会显示“已连接”。****

    :::image type="content" source="media/analysis-services-gateway-install/aas-gateway-connect-success.png" alt-text="Select":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

使用 [Get-AzResource](https://docs.microsoft.com/powershell/module/az.resources/get-azresource) 获取网关 ResourceID。 然后，通过在 [Set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver) 或 [New-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/new-azanalysisservicesserver) 中指定“-GatewayResourceID”，将网关资源连接到现有服务器或新服务器。

若要获取网关资源 ID：

```powershell
Connect-AzAccount -Environment AzureChinaCloud -Tenant $TenantId -Subscription $subscriptionIdforGateway -Environment "AzureCloud"
$GatewayResourceId = $(Get-AzResource -ResourceType "Microsoft.Web/connectionGateways" -Name $gatewayName).ResourceId  

```

若要配置现有服务器：

```powershell
Connect-AzAccount -Environment AzureChinaCloud -Tenant $TenantId -Subscription $subscriptionIdforAzureAS -Environment "AzureCloud"
Set-AzAnalysisServicesServer -ResourceGroupName $RGName -Name $servername -GatewayResourceId $GatewayResourceId

```
---

就这么简单。 如果你需要打开端口或执行任何故障排除时，一定要签出[本地数据网关](analysis-services-gateway.md)。

## <a name="next-steps"></a>后续步骤

* [管理 Analysis Services](analysis-services-manage.md)   
* [从 Azure Analysis Services 中获取数据](analysis-services-connect.md)   
* [对 Azure 虚拟网络上的数据源使用网关](analysis-services-vnet-gateway.md)

<!-- Update_Description: update meta properties, wording update, update link -->