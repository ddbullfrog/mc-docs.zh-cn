---
title: 重置有故障的 Azure ExpressRoute 线路：PowerShell
description: 本文帮助你重置处于故障状态的 ExpressRoute 线路。
documentationcenter: na
services: expressroute
author: anzaman
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 11/28/2017
ms.date: 04/01/2018
ms.author: v-yiso
ms.openlocfilehash: aa40a91acabf3280b2732b3b0e5096057a920d8f
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327952"
---
# <a name="reset-a-failed-expressroute-circuit"></a>重置有故障的 ExpressRoute 线路

针对 ExpressRoute 线路执行的某个操作未成功完成时，该线路可能进入“故障”状态。 本文帮助你重置有故障的 Azure ExpressRoute 线路。

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="reset-a-circuit"></a>重置线路

1. 安装最新版本的 Azure 资源管理器 PowerShell cmdlet。 有关详细信息，请参阅[安装和配置 Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps)。

2. 使用提升的权限打开 PowerShell 控制台，并连接到帐户。 使用下面的示例来帮助连接：

   ```azurepowershell
   Connect-AzAccount
   ```
3. 如果有多个 Azure 订阅，请查看该帐户的订阅。

   ```azurepowershell
   Get-AzSubscription
   ```
4. 指定要使用的订阅。

   ```azurepowershell
   Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
   ```
5. 运行以下命令可重置处于故障状态的线路：

   ```azurepowershell
   $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

现在，该线路应已恢复正常。 如果线路仍处于故障状态，请向 [Microsoft 支持部门](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)开具支持票证。

## <a name="next-steps"></a>后续步骤

如果仍然存在问题，请通过 [Microsoft 支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)开具一个支持票证。
