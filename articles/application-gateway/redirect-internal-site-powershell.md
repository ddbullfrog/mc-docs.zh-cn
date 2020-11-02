---
title: 使用 PowerShell 进行内部重定向
titleSuffix: Azure Application Gateway
description: 了解如何使用 Azure Powershell 创建将内部 Web 流量重定向到相应的后端服务器池的应用程序网关。
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 10/22/2020
ms.author: v-junlch
ms.openlocfilehash: 4274902564367c6af7a61de4ee809295470b3c0e
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472590"
---
# <a name="create-an-application-gateway-with-internal-redirection-using-azure-powershell"></a>使用 Azure PowerShell 创建支持内部重定向的应用程序网关

你可以使用 Azure Powershell 配置 [web 流量重定向](multiple-site-overview.md)创建时[应用程序网关](overview.md)。 在本文中，将使用虚拟机规模集定义后端池。 然后，基于所拥有的域配置侦听器和规则，以确保 Web 流量可到达相应池。 本文假定你拥有多个域，并使用示例 www\.contoso.com 和 www\.contoso.org 。

在本文中，学习如何：

* 设置网络
* 创建应用程序网关
* 添加侦听器和重定向规则
* 使用后端池创建虚拟机规模集
* 在域中创建 CNAME 记录

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

如果选择在本地安装并使用 PowerShell，则本文需要 Azure PowerShell 模块 1.0.0 或更高版本。 若要查找版本，请运行 `Get-Module -ListAvailable Az`。 如果需要进行升级，请参阅 [Install Azure PowerShell module](https://docs.microsoft.com/powershell/azure/install-az-ps)（安装 Azure PowerShell 模块）。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 来创建与 Azure 的连接。

## <a name="create-a-resource-group"></a>创建资源组

资源组是在其中部署和管理 Azure 资源的逻辑容器。 使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建 Azure 资源组。  

```azurepowershell
New-AzResourceGroup -Name myResourceGroupAG -Location chinanorth2
```

## <a name="create-network-resources"></a>创建网络资源

使用 [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig) 创建 *myBackendSubnet* 和 *myAGSubnet* 的子网配置。 使用 [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) 和子网配置创建名为 *myVNet* 的虚拟网络。 最后使用 [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) 创建名为 *myAGPublicIPAddress* 的公共 IP 地址。 这些资源用于提供与应用程序网关及其关联资源的网络连接。

```azurepowershell
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24
$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.2.0/24
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location chinanorth2 `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $backendSubnetConfig, $agSubnetConfig
$pip = New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location chinanorth2 `
  -Name myAGPublicIPAddress `
  -AllocationMethod Dynamic
```

## <a name="create-an-application-gateway"></a>创建应用程序网关

### <a name="create-the-ip-configurations-and-frontend-port"></a>创建 IP 配置和前端端口

使用 [New-AzApplicationGatewayIPConfiguration](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewayipconfiguration) 将前面创建的 *myAGSubnet* 关联到应用程序网关。 使用 [New-AzApplicationGatewayFrontendIPConfig](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig) 将 *myAGPublicIPAddress* 分配给应用程序网关。 然后，可以使用 [New-AzApplicationGatewayFrontendPort](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewayfrontendport) 创建 HTTP 端口。

```azurepowershell
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$subnet=$vnet.Subnets[0]
$gipconfig = New-AzApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet
$fipconfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip
$frontendPort = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 80
```

### <a name="create-the-backend-pool-and-settings"></a>创建后端池和设置

使用 [New-AzApplicationGatewayBackendAddressPool](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool) 为应用程序网关创建名为 *contosoPool* 的后端池。 使用 [New-AzApplicationGatewayBackendHttpSettings](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting) 配置后端池的设置。

```azurepowershell
$contosoPool = New-AzApplicationGatewayBackendAddressPool `
  -Name contosoPool 
$poolSettings = New-AzApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-first-listener-and-rule"></a>创建第一个侦听器和规则

应用程序网关需要侦听器才能适当地将流量路由到后端池。 在本文中，将为两个域创建两个侦听器。 在此示例中，将为域 *www\.contoso.com* 和 *www\.contoso.org* 创建侦听器。

使用 [New-AzApplicationGatewayHttpListener](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewayhttplistener) 以及前面创建的前端配置和前端端口创建名为 *contosoComListener* 的第一个侦听器。 侦听器需要使用规则来了解哪个后端池使用传入流量。 使用 [New-AzApplicationGatewayRequestRoutingRule](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule) 创建一个名为 *contosoComRule* 的基本规则。

```azurepowershell
$contosoComlistener = New-AzApplicationGatewayHttpListener `
  -Name contosoComListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendPort `
  -HostName "www.contoso.com"
$frontendRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name contosoComRule `
  -RuleType Basic `
  -HttpListener $contosoComListener `
  -BackendAddressPool $contosoPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>创建应用程序网关

现在已创建所需的支持资源，请使用 [New-AzApplicationGatewaySku](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgatewaysku) 为名为 *myAppGateway* 的应用程序网关指定参数，然后再使用 [New-AzApplicationGateway](https://docs.microsoft.com/powershell/module/az.network/new-azapplicationgateway) 创建它。

```azurepowershell
$sku = New-AzApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2
$appgw = New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location chinanorth2 `
  -BackendAddressPools $contosoPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendPort `
  -HttpListeners $contosoComListener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku
```

### <a name="add-the-second-listener"></a>添加第二个侦听器

使用 [Add-AzApplicationGatewayHttpListener](https://docs.microsoft.com/powershell/module/az.network/add-azapplicationgatewayhttplistener) 添加重定向流量所需的名为 *contosoOrgListener* 的侦听器。

```azurepowershell
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$frontendPort = Get-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -ApplicationGateway $appgw
$ipconfig = Get-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayHttpListener `
  -ApplicationGateway $appgw `
  -Name contosoOrgListener `
  -Protocol Http `
  -FrontendIPConfiguration $ipconfig `
  -FrontendPort $frontendPort `
  -HostName "www.contoso.org"
Set-AzApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-the-redirection-configuration"></a>添加重定向配置

可以使用 [Add-AzApplicationGatewayRedirectConfiguration](https://docs.microsoft.com/powershell/module/az.network/add-azapplicationgatewayredirectconfiguration) 为侦听器配置重定向。 

```azurepowershell
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$contosoComlistener = Get-AzApplicationGatewayHttpListener `
  -Name contosoComListener `
  -ApplicationGateway $appgw
$contosoOrglistener = Get-AzApplicationGatewayHttpListener `
  -Name contosoOrgListener `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayRedirectConfiguration `
  -ApplicationGateway $appgw `
  -Name redirectOrgtoCom `
  -RedirectType Found `
  -TargetListener $contosoComListener `
  -IncludePath $true `
  -IncludeQueryString $true
Set-AzApplicationGateway -ApplicationGateway $appgw
```

### <a name="add-the-second-routing-rule"></a>添加第二个路由规则

然后可以使用 [Add-AzApplicationGatewayRequestRoutingRule](https://docs.microsoft.com/powershell/module/az.network/add-azapplicationgatewayrequestroutingrule) 将重定向配置关联到名为 *contosoOrgRule* 的新规则。

```azurepowershell
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$contosoOrglistener = Get-AzApplicationGatewayHttpListener `
  -Name contosoOrgListener `
  -ApplicationGateway $appgw
$redirectConfig = Get-AzApplicationGatewayRedirectConfiguration `
  -Name redirectOrgtoCom `
  -ApplicationGateway $appgw   
Add-AzApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $appgw `
  -Name contosoOrgRule `
  -RuleType Basic `
  -HttpListener $contosoOrgListener `
  -RedirectConfiguration $redirectConfig
Set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="create-a-virtual-machine-scale-set"></a>创建虚拟机规模集

在此示例中，将创建一个虚拟机规模集以支持所创建的后端池。 创建的规模集名为 *myvmss* ，并包含两个在其上安装了 IIS 的虚拟机实例。 配置 IP 设置时将规模集分配给后端池。

```azurepowershell
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$backendPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name contosoPool `
  -ApplicationGateway $appgw
$ipConfig = New-AzVmssIpConfig `
  -Name myVmssIPConfig `
  -SubnetId $vnet.Subnets[1].Id `
  -ApplicationGatewayBackendAddressPoolsId $backendPool.Id
$vmssConfig = New-AzVmssConfig `
  -Location chinanorth2 `
  -SkuCapacity 2 `
  -SkuName Standard_DS2 `
  -UpgradePolicyMode Automatic
Set-AzVmssStorageProfile $vmssConfig `
  -ImageReferencePublisher MicrosoftWindowsServer `
  -ImageReferenceOffer WindowsServer `
  -ImageReferenceSku 2016-Datacenter `
  -ImageReferenceVersion latest `
  -OsDiskCreateOption FromImage
Set-AzVmssOsProfile $vmssConfig `
  -AdminUsername azureuser `
  -AdminPassword "Azure123456!" `
  -ComputerNamePrefix myvmss
Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name myVmssNetConfig `
  -Primary $true `
  -IPConfiguration $ipConfig
New-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmssConfig
```

### <a name="install-iis"></a>安装 IIS

```azurepowershell
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
$vmss = Get-AzVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings
Update-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmss
```

## <a name="create-cname-record-in-your-domain"></a>在域中创建 CNAME 记录

使用其公共 IP 地址创建应用程序网关后，可以获取 DNS 地址并使用它在域中创建 CNAME 记录。 可以使用 [Get-AzPublicIPAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress) 获取应用程序网关的 DNS 地址。 复制 DNSSettings 的 *fqdn* 值并使用它作为所创建的 CNAME 记录的值。 不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。

```azurepowershell
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

## <a name="test-the-application-gateway"></a>测试应用程序网关

在浏览器的地址栏中输入域名。 例如， `https://www.contoso.com`。

![在应用程序网关中测试 contoso 站点](./media/redirect-internal-site-powershell/application-gateway-iistest.png)

将地址更改为其他域（例如 `https://www.contoso.org`），应会看到流量已被重定向回 www\.contoso.com 的侦听器。

## <a name="next-steps"></a>后续步骤

- [应用程序网关重定向概述](redirect-overview.md)

