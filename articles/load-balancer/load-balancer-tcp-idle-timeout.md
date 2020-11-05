---
title: 在 Azure 中配置负载均衡器 TCP 重置和空闲超时
titleSuffix: Azure Load Balancer
description: 本文介绍如何配置 Azure 负载均衡器 TCP 空闲超时。
services: load-balancer
documentationcenter: na
author: WenJason
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 10/09/2020
ms.date: 11/02/2020
ms.author: v-jay
ms.openlocfilehash: fbaff92dd13b3276474ba18bc34546eaea96f9b1
ms.sourcegitcommit: 1f933e4790b799ceedc685a0cea80b1f1c595f3d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92628258"
---
# <a name="configure-tcp-idle-timeout-for-azure-load-balancer"></a>配置 Azure 负载均衡器的 TCP 空闲超时

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

本文需要 Azure PowerShell 模块 5.4.1 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要进行升级，请参阅 [Install Azure PowerShell module](https://docs.microsoft.com/powershell/azure/install-Az-ps)（安装 Azure PowerShell 模块）。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 以创建与 Azure 的连接。

Azure 负载均衡器的空闲超时设置为 4 分钟到 120 分钟。 默认情况下，它设置为 4 分钟。 如果处于非活动状态的时间超过超时值，则不能保证在客户端和云服务之间保持 TCP 或 HTTP 会话。 详细了解 [TCP 空闲超时](load-balancer-tcp-reset.md)。

以下部分介绍如何更改公共 IP 和负载均衡器资源的空闲超时设置。


## <a name="configure-the-tcp-idle-timeout-for-your-public-ip"></a>配置公共 IP 的 TCP 空闲超时

```azurepowershell
$publicIP = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyResourceGroup
$publicIP.IdleTimeoutInMinutes = "15"
Set-AzPublicIpAddress -PublicIpAddress $publicIP
```

`IdleTimeoutInMinutes` 是可选项。 如果未设置，默认超时为 4 分钟。 可接受的超时范围为 4 到 120 分钟。

## <a name="set-the-tcp-idle-timeout-on-rules"></a>在规则上设置 TCP 空闲超时

若要为负载均衡器设置空闲超时，请在负载均衡规则上设置“IdleTimeoutInMinutes”。 例如：

```azurepowershell
$lb = Get-AzLoadBalancer -Name "MyLoadBalancer" -ResourceGroup "MyResourceGroup"
$lb | Set-AzLoadBalancerRuleConfig -Name myLBrule -IdleTimeoutInMinutes 15
```

## <a name="next-steps"></a>后续步骤

[内部负载均衡器概述](load-balancer-internal-overview.md)

[开始配置面向 Internet 的负载均衡器](quickstart-load-balancer-standard-public-powershell.md)

[配置负载均衡器分发模式](load-balancer-distribution-mode.md)
