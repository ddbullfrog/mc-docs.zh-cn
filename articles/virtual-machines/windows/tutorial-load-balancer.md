---
title: 教程 - 在 Azure 中对 Windows 虚拟机进行负载均衡
description: 本教程介绍如何使用 Azure PowerShell 创建负载均衡器，以实现在三个 Windows 虚拟机上高度可用且安全的应用程序
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
origin.date: 12/03/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: dbed864abef9c8095c4109f84db5ad466efab725
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105684"
---
# <a name="tutorial-load-balance-windows-virtual-machines-in-azure-to-create-a-highly-available-application-with-azure-powershell"></a>教程：在 Azure 中使用 Azure PowerShell 均衡 Windows 虚拟机负载以创建高可用性应用程序
负载均衡通过将传入请求分布到多个虚拟机来提供更高级别的可用性。 本教程介绍了 Azure 负载均衡器的不同组件，这些组件用于分发流量和提供高可用性。 你将学习如何执行以下操作：

> [!div class="checklist"]
> * 创建 Azure 负载均衡器
> * 创建负载均衡器运行状况探测
> * 创建负载均衡器流量规则
> * 使用自定义脚本扩展创建基本的 IIS 站点
> * 创建虚拟机并将其附加到负载均衡器
> * 查看负载均衡器的实际运行情况
> * 在负载均衡器中添加和删除 VM

## <a name="azure-load-balancer-overview"></a>Azure 负载均衡器概述
Azure 负载均衡器是位于第 4 层（TCP、UDP）的负载均衡器，通过在正常运行的 VM 之间分发传入流量提供高可用性。 负载均衡器运行状况探测器监视每个 VM 上的给定端口，仅将流量分发给正常运行的 VM。

定义包含一个或多个公共 IP 地址的前端 IP 配置。 利用此前端 IP 配置，可通过 Internet 访问负载均衡器和应用程序。 

虚拟机使用其虚拟网络接口卡 (NIC) 连接到负载均衡器。 若要向 VM 分发流量，后端地址池需包含连接到负载均衡器的虚拟 NIC 的 IP 地址。

若要控制流量流，需为映射到 VM 的特定端口和协议定义负载均衡器规则。

## <a name="launch-azure-local-shell"></a>启动 Azure 本地 Shell

打开 Azure Powershell 控制台，并以管理员权限运行以下脚本。

<!--Not Available on Azure Cloud Shell-->

## <a name="create-azure-load-balancer"></a>创建 Azure 负载均衡器
本部分详细介绍如何创建和配置负载均衡器的每个组件。 创建负载均衡器之前，需使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建资源组。 以下示例在 ChinaEast 位置创建名为 myResourceGroupLoadBalancer 的资源组：

```powershell
New-AzResourceGroup `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "ChinaEast"
```

### <a name="create-a-public-ip-address"></a>创建公共 IP 地址
若要通过 Internet 访问应用，需要负载均衡器的一个公共 IP 地址。 使用 [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) 创建一个公共 IP 地址。 以下示例在 myResourceGroupLoadBalancer 资源组中创建名为 myPublicIP 的公共 IP 地址：

```powershell
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "ChinaEast" `
  -AllocationMethod "Static" `
  -Name "myPublicIP"
```

### <a name="create-a-load-balancer"></a>创建负载均衡器
使用 [New-AzLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerfrontendipconfig) 创建一个前端 IP 池。 以下示例创建名为 *myFrontEndPool* 的前端 IP 池并附加 *myPublicIP* 地址： 

```powershell
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
```

使用 [New-AzLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) 创建一个后端地址池。 在剩余的步骤中，各个 VM 将附加到此后端池。 以下示例创建名为 myBackEndPool 的后端地址池：

```powershell
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig `
  -Name "myBackEndPool"
```

现在，使用 [New-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancer) 创建负载均衡器。 以下示例使用在前面的步骤中创建的前端和后端 IP 池创建名为 *myLoadBalancer* 的负载均衡器：

```powershell
$lb = New-AzLoadBalancer `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myLoadBalancer" `
  -Location "ChinaEast" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool
```

### <a name="create-a-health-probe"></a>创建运行状况探测器
若要允许负载均衡器监视应用的状态，可以使用运行状况探测器。 运行状况探测器基于其对运行状况检查的响应，从负载均衡器中动态添加或删除 VM。 默认情况下，在 15 秒时间间隔内发生两次连续的故障后，会从负载均衡器分布中删除 VM。 可以为应用创建基于协议或特定运行状况检查页面的运行状况探测器。 

以下示例创建一个 TCP 探测器。 还可创建自定义 HTTP 探测器，以便执行更精细的运行状况检查。 使用自定义 HTTP 探测器时，必须创建运行状况检查页，例如 healthcheck.aspx。 探测器必须为负载均衡器返回 HTTP 200 OK 响应，以保持主机处于旋转状态。

若要创建 TCP 运行状况探测，请使用 [Add-AzLoadBalancerProbeConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerprobeconfig)。 以下示例创建名为 *myHealthProbe* 的运行状况探测器，用于在 *TCP* 端口 *80* 上监视每个 VM：

```powershell
Add-AzLoadBalancerProbeConfig `
  -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol tcp `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
```

若要应用运行状况探测，请使用 [Set-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/set-azloadbalancer) 更新负载均衡器：

```powershell
Set-AzLoadBalancer -LoadBalancer $lb
```

### <a name="create-a-load-balancer-rule"></a>创建负载均衡器规则
负载均衡器规则用于定义将流量分配给 VM 的方式。 定义传入流量的前端 IP 配置和后端 IP 池以接收流量，同时定义所需源和目标端口。 若要确保仅正常运行的 VM 接收流量，还需定义要使用的运行状况探测。

使用 [Add-AzLoadBalancerRuleConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerruleconfig) 创建一个负载均衡器规则。 以下示例创建名为 *myLoadBalancerRule* 的负载均衡器规则并均衡 *TCP* 端口 *80* 上的流量：

```powershell
$probe = Get-AzLoadBalancerProbeConfig -LoadBalancer $lb -Name "myHealthProbe"

Add-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe $probe
```

使用 [Set-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/set-azloadbalancer) 更新负载均衡器：

```powershell
Set-AzLoadBalancer -LoadBalancer $lb
```

## <a name="configure-virtual-network"></a>配置虚拟网络
需要先创建提供支持的虚拟网络资源，然后才能部署某些 VM 并测试均衡器。 有关虚拟网络的详细信息，请参阅[管理 Azure 虚拟网络](tutorial-virtual-network.md)教程。

### <a name="create-network-resources"></a>创建网络资源
使用 [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络。 以下示例创建包含 mySubnet 的名为 myVnet 的虚拟网络：

```powershell
# Create subnet config
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "ChinaEast" `
  -Name "myVnet" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig
```

使用 [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface) 创建虚拟 NIC。 以下示例创建三个虚拟 NIC。 （在以下步骤中针对为应用创建的每个 VM 各使用一个虚拟 NIC）。 可随时创建其他虚拟 NIC 和 VM，并将其添加到负载均衡器：

```powershell
for ($i=1; $i -le 3; $i++)
{
   New-AzNetworkInterface `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -Name myVM$i `
     -Location "ChinaEast" `
     -Subnet $vnet.Subnets[0] `
     -LoadBalancerBackendAddressPool $lb.BackendAddressPools[0]
}
```

## <a name="create-virtual-machines"></a>创建虚拟机
若要提高应用的高可用性，请将 VM 放置在可用性集中。

使用 [New-AzAvailabilitySet](https://docs.microsoft.com/powershell/module/az.compute/new-azavailabilityset) 创建一个可用性集。 以下示例创建名为“myAvailabilitySet”  的可用性集：

```powershell
$availabilitySet = New-AzAvailabilitySet `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myAvailabilitySet" `
  -Location "ChinaEast" `
  -Sku aligned `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2
```

使用 [New-AzureRmNetworkInterface](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) 设置 VM 的管理员用户名和密码：

```powershell
$cred = Get-Credential
```

现在，可使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 以下示例创建三个 VM 和所需的虚拟网络组件（如果它们尚不存在）：

```powershell
for ($i=1; $i -le 3; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupLoadBalancer" `
        -Name "myVM$i" `
        -Location "China East" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -OpenPorts 80 `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred `
        -AsJob
}
```

`-AsJob` 参数以后台任务的方式创建 VM，因此 PowerShell 提示符会返回到你所在的位置。 可以通过 `Job` cmdlet 查看后台作业的详细信息。 创建和配置所有三个 VM 需要几分钟时间。

### <a name="install-iis-with-custom-script-extension"></a>使用自定义脚本扩展安装 IIS
在有关[如何自定义 Windows 虚拟机](tutorial-automate-vm-deployment.md)的上一教程中，你已了解如何使用 Windows 的自定义脚本扩展自动执行 VM 自定义。 可使用相同的方法在 VM 上安装和配置 IIS。

使用 [Set-AzVMExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmextension) 安装自定义脚本扩展。 该扩展运行 `powershell Add-WindowsFeature Web-Server` 来安装 IIS Web 服务器，然后更新 Default.htm 页以显示 VM 的主机名：

```powershell
for ($i=1; $i -le 3; $i++)
{
   Set-AzVMExtension `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -ExtensionName "IIS" `
     -VMName myVM$i `
     -Publisher Microsoft.Compute `
     -ExtensionType CustomScriptExtension `
     -TypeHandlerVersion 1.8 `
     -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
     -Location ChinaEast
}
```

## <a name="test-load-balancer"></a>测试负载均衡器
使用 [Get-AzPublicIPAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress) 获取负载均衡器的公共 IP 地址。 以下示例获取前面创建的“myPublicIP”  的 IP 地址：

```powershell
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myPublicIP" | select IpAddress
```

然后，可将公共 IP 地址输入 Web 浏览器中。 网站随即显示，其中包括负载均衡器将流量分发到的 VM 的主机名，如下例所示：

:::image type="content" source="./media/tutorial-load-balancer/running-iis-website.png" alt-text="运行 IIS 网站":::

若要查看负载均衡器如何在运行应用的所有 3 个 VM 之间分配流量，可强制刷新 Web 浏览器。

## <a name="add-and-remove-vms"></a>添加和删除 VM
建议对运行应用的 VM 执行维护，例如安装 OS 更新。 若要应对应用增加的流量，建议添加更多 VM。 本部分演示了如何在负载均衡器中删除或添加 VM。

### <a name="remove-a-vm-from-the-load-balancer"></a>从负载均衡器中删除 VM
使用 [Get-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/get-aznetworkinterface) 获取网络接口卡，然后将虚拟 NIC 的 LoadBalancerBackendAddressPools 属性设置为“$null”。 最后，更新虚拟 NIC：

```powershell
$nic = Get-AzNetworkInterface `
    -ResourceGroupName "myResourceGroupLoadBalancer" `
    -Name "myVM2"
$nic.Ipconfigurations[0].LoadBalancerBackendAddressPools=$null
Set-AzNetworkInterface -NetworkInterface $nic
```

若要查看负载均衡器如何在运行应用的其余两个 VM 之间分发流量，可强制刷新 web 浏览器。 现在可以对 VM 执行维护，例如安装 OS 更新或执行 VM 重新启动。

### <a name="add-a-vm-to-the-load-balancer"></a>将 VM 添加到负载均衡器
执行 VM 维护后，或者如果需要扩展容量，请通过 [Get-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/get-azloadbalancer) 将虚拟 NIC 的 LoadBalancerBackendAddressPools 属性设置为“BackendAddressPool”：

获取负载均衡器：

```powershell
$lb = Get-AzLoadBalancer `
    -ResourceGroupName myResourceGroupLoadBalancer `
    -Name myLoadBalancer 
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$lb.BackendAddressPools[0]
Set-AzNetworkInterface -NetworkInterface $nic
```

## <a name="next-steps"></a>后续步骤

在本教程中，你已创建了一个负载均衡器并已将 VM 附加到它。 你已了解如何执行以下操作：

> [!div class="checklist"]
> * 创建 Azure 负载均衡器
> * 创建负载均衡器运行状况探测
> * 创建负载均衡器流量规则
> * 使用自定义脚本扩展创建基本的 IIS 站点
> * 创建虚拟机并将其附加到负载均衡器
> * 查看负载均衡器的实际运行情况
> * 在负载均衡器中添加和删除 VM

请转到下一教程，了解如何管理 VM 网络。

> [!div class="nextstepaction"]
> [管理 VM 和虚拟网络](./tutorial-virtual-network.md)

<!-- Update_Description: update meta properties, wording update, update link -->