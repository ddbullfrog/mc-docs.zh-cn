---
title: 创建公共 IP - Azure PowerShell
description: 了解如何使用 Azure PowerShell 创建公共 IP
services: virtual-network
documentationcenter: na
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 08/28/2020
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: yes
ms.testdate: 10/05/2020
ms.author: v-yeche
ms.openlocfilehash: e778c82127f234e05862cc7b64c8eef0572e3ad8
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91571582"
---
<!--Verified Successfully-->
<!--Remove the part of Availability Zones-->
# <a name="quickstart-create-a-public-ip-address-using-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建公共 IP 地址

本文介绍如何使用 Azure PowerShell 来创建公共 IP 地址资源。 若要详细了解此操作可以关联到的具体资源、基本 SKU 和标准 SKU 之间的差异，以及其他相关信息，请参阅[公共 IP 地址](/virtual-network/public-ip-addresses)。  对于此示例，我们将仅重点介绍 IPv4 地址；有关 IPv6 地址的详细信息，请参阅[适用于 Azure VNet 的 IPv6](/virtual-network/ipv6-overview)。

## <a name="prerequisites"></a>先决条件

- 本地安装的 Azure PowerShell 或 Azure 本地 Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

<!--Not Available on Azure Cloud Shell-->

如果选择在本地安装并使用 PowerShell，则本文需要 Azure PowerShell 模块 5.4.1 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 以创建与 Azure 的连接。

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 在 chinaeast2 位置创建名为“myResourceGroup”的资源组 。

```powershell
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'chinaeast2'

New-AzResourceGroup -Name $rg -Location $loc
```

<!--Not Available on Availability Zones-->
<!--Not Available on Availability Zones-->

<a name="option-create-public-ip-basic"></a>
## <a name="basic-sku"></a>基本 SKU

使用 [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) 在 myResourceGroup 中创建名为“myStandardPublicIP”的基本静态公共 IP 地址。  

<!--Not Available on Availability Zones-->

```powershell
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'chinaeast2'
$pubIP = 'myBasicPublicIP'
$sku = 'Basic'
$alloc = 'Static'

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku
```
如果可以接受 IP 地址随时间而变这种情况，可通过将 AllocationMethod 更改为“Dynamic”来选择动态 IP 分配。

---

## <a name="additional-information"></a>其他信息 

若要更详细地了解以上所列各个变量，请参阅[管理公共 IP 地址](/virtual-network/virtual-network-public-ip-address#create-a-public-ip-address)。

## <a name="next-steps"></a>后续步骤
- [将公共 IP 地址关联到虚拟机](/virtual-network/associate-public-ip-address-vm#azure-portal)
- 详细了解 Azure 中的[公共 IP 地址](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses)。
- 详细了解所有[公共 IP 地址设置](virtual-network-public-ip-address.md#create-a-public-ip-address)。

<!-- Update_Description: new article about create public ip powershell -->
<!--NEW.date: 10/05/2020-->