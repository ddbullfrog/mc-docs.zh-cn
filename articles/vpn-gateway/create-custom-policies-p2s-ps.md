---
title: 为点到站点创建和设置自定义 IPsec 策略：PowerShell
titleSuffix: Azure VPN Gateway
description: 本文可帮助你为 VPN 网关 P2S 配置创建和设置自定义 IPSec 策略
services: vpn-gateway
author: WenJason
ms.service: vpn-gateway
ms.topic: how-to
origin.date: 09/09/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: 53bccff9e29c9734d19c02eef6e9eefd3939c660
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395570"
---
# <a name="create-and-set-custom-ipsec-policies-for-point-to-site"></a>为点到站点创建和设置自定义 IPsec 策略

如果你的环境需要自定义 IPsec 策略来用于加密，则可以轻松地使用所需设置来配置策略对象。 本文可帮助你创建自定义策略对象，然后使用 PowerShell 对其进行设置。

## <a name="before-you-begin"></a>准备阶段

### <a name="prerequisites"></a>先决条件

验证你的环境是否满足以下先决条件：

* 你已配置正常运行的点到站点 VPN。 如果未配置，请使用 [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md) 或 [Azure 门户](vpn-gateway-howto-point-to-site-resource-manager-portal.md)，按照“创建点到站点 VPN”一文中的步骤创建一个。

### <a name="working-with-azure-powershell"></a>使用 Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-powershell-locally.md)]

## <a name="1-set-variables"></a><a name="signin"></a>1.设置变量

声明要使用的变量。 使用以下示例，根据需要将值替换为自己的值。 如果在练习期间的任何时候关闭了 PowerShell 会话，只需再次复制和粘贴这些值，以重新声明变量。

  ```azurepowershell
  $RG = "TestRG"
  $GWName = "VNet1GW"
  ```

## <a name="2-create-policy-object"></a><a name="create"></a>2.创建策略对象

创建自定义 IPsec 策略对象。 可以调整值以满足所需的条件。

```azurepowershell
$vpnclientipsecpolicy = New-AzVpnClientIpsecPolicy -IpsecEncryption AES256 -IpsecIntegrity SHA256 -SALifeTime 86471 -SADataSize 429496 -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup2 -PfsGroup PFS2
```

## <a name="3-update-gateway-and-set-policy"></a><a name="update"></a>3.更新网关并设置策略

在此步骤中，更新现有 P2S VPN 网关，并设置 IPsec 策略。

```azurepowershell
$gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gateway -VpnClientIpsecPolicy $vpnclientipsecpolicy
```

## <a name="next-steps"></a>后续步骤

有关 P2S 配置的详细信息，请参阅[关于点到站点 VPN](point-to-site-about.md)。