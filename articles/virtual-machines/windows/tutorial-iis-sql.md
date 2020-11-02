---
title: 教程 - 在 Azure 中创建运行 SQL、IIS、.NET 堆栈的 VM
description: 本教程介绍如何在 Azure 中的 Windows 虚拟机上安装 Azure SQL、IIS、.NET 堆栈。
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
ms.openlocfilehash: 6ef0249bbdc15cfbfaa00a2280dff4873ced5024
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105724"
---
# <a name="tutorial-install-the-sql-iis-net-stack-in-a-windows-vm-with-azure-powershell"></a>教程：使用 Azure PowerShell 在 Windows VM 中安装 SQL、IIS、.NET 堆栈

在本教程中，我们将使用 Azure PowerShell 安装 SQL、IIS、.NET 堆栈。 此堆栈包含两个运行 Windows Server 2016 的 VM，一个带有 IIS 和 .NET，另一个带有 SQL Server。

> [!div class="checklist"]
> * 创建 VM 
> * 在 VM 上安装 IIS 和 .NET Core SDK
> * 创建运行 SQL Server 的 VM
> * 安装 SQL Server 扩展

## <a name="launch-azure-local-shell"></a>启动 Azure 本地 Shell

打开 Azure Powershell 控制台，并以管理员权限运行以下脚本。

<!--Not Available on Azure Cloud Shell-->

## <a name="create-an-iis-vm"></a>创建 IIS VM 

在此示例中，我们使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) cmdlet 在 PowerShell 中快速创建 Windows Server 2016 Datacenter，然后安装 IIS 和 .NET Framework。 IIS 和 SQL VM 共享资源组和虚拟网络，因此我们创建这些名称的变量。

<!--Notice: Remove Cloud Shell-->

```powershell
Connect-AzAccount -Environment AzureChinaCloud
$vmName = "IISVM"
$vNetName = "myIISSQLvNet"
$resourceGroup = "myIISSQLGroup"
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name $vmName `
    -Location "China East" `
    -VirtualNetworkName $vNetName `
    -SubnetName "myIISSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -AddressPrefix 192.168.0.0/16 `
    -PublicIpAddressName "myIISPublicIpAddress" `
    -OpenPorts 80,3389 
```

通过 [Set-AzVMExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmextension) cmdlet 使用自定义脚本扩展安装 IIS 和 .NET Framework。

```powershell
Set-AzVMExtension `
    -ResourceGroupName $resourceGroup `
    -ExtensionName IIS `
    -VMName $vmName `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server,Web-Asp-Net45,NET-Framework-Features"}' `
    -Location ChinaEast
```

## <a name="create-another-subnet"></a>创建另一子网

<!--Correct on [Get-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetwork).-->

为 SQL VM 创建第二个子网。 使用 [Get-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetwork) 获取 vNet。

```powershell
$vNet = Get-AzVirtualNetwork `
   -Name $vNetName `
   -ResourceGroupName $resourceGroup
```

使用 [Add-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/add-azvirtualnetworksubnetconfig) 为子网创建配置。

```powershell
Add-AzVirtualNetworkSubnetConfig `
   -AddressPrefix 192.168.0.0/24 `
   -Name mySQLSubnet `
   -VirtualNetwork $vNet `
   -ServiceEndpoint Microsoft.Sql
```

通过 [Set-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/set-azvirtualnetwork) 使用新的子网信息更新 vNet

```powershell   
$vNet | Set-AzVirtualNetwork
```

## <a name="azure-sql-vm"></a>Azure SQL VM

使用 SQL Server 的预配置 Azure 市场映像创建 SQL VM。 首先创建 VM，然后在 VM 上安装 SQL Server 扩展。 

```powershell
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name "mySQLVM" `
    -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
    -Location chinaeast `
    -VirtualNetworkName $vNetName `
    -SubnetName "mySQLSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "mySQLPublicIpAddress" `
    -OpenPorts 3389,1401 
```

使用 [Set-AzVMSqlServerExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmsqlserverextension) 将 [SQL Server 扩展](../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md)添加到 SQL VM。

```powershell
Set-AzVMSqlServerExtension `
   -ResourceGroupName $resourceGroup  `
   -VMName mySQLVM `
   -Name "SQLExtension" `
   -Location "ChinaEast"
```

## <a name="next-steps"></a>后续步骤

在本教程中，已使用 Azure PowerShell 安装 SQL&#92;IIS&#92;.NET 堆栈。 你已了解如何：

> [!div class="checklist"]
> * 创建 VM 
> * 在 VM 上安装 IIS 和 .NET Core SDK
> * 创建运行 SQL Server 的 VM
> * 安装 SQL Server 扩展

转到下一教程，了解如何使用 TLS/SSL 证书保护 IIS Web 服务器。

> [!div class="nextstepaction"]
> [使用 TLS/SSL 证书保护 IIS Web 服务器](tutorial-secure-web-server.md)

<!-- Update_Description: update meta properties, wording update, update link -->