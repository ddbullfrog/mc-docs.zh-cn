---
title: 教程：配置用于 Microsoft 对等互连的路由筛选器 - Azure PowerShell
description: 本教程介绍如何使用 PowerShell 配置用于 Microsoft 对等互连的路由筛选器。
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
origin.date: 10/08/2020
ms.date: 11/16/2020
ms.author: v-yiso
ms.custom: seodec18
ms.openlocfilehash: db1a4b9036bbfddc58d9f68e76ed6e3cb07400a2
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328141"
---
# <a name="tutorial-configure-route-filters-for-microsoft-peering-using-powershell"></a>教程：使用 PowerShell 配置用于 Microsoft 对等互连的路由筛选器

> [!div class="op_single_selector"]
> * [Azure 门户](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [Azure CLI](how-to-routefilter-cli.md)
> 

路由筛选器是通过 Microsoft 对等互连使用部分受支持服务的一种方法。 本文中的步骤可帮助配置和管理 ExpressRoute 线路的路由筛选器。

可以通过 Microsoft 对等互连访问 Azure 公共服务（如存储和 SQL DB）。 Azure 公共服务可按区域选择，但不能针对每个公共服务定义。

如果在 ExpressRoute 线路中配置了 Microsoft 对等互连并附加了路由筛选器，则会通过建立的 BGP 会话播发为这些服务选择的所有前缀。 每个前缀附加有 BGP 团体值，以标识通过该前缀提供的服务。 如需连接所有服务，则应通过 BGP 播发大量前缀。 这会显著增加网络中路由器所维护路由表的大小。 如果打算仅使用通过 Microsoft 对等互连提供的一部分服务，可通过两种方式减少路由表大小。 方法：

* 通过在 BGP 团体上应用路由筛选器，筛选出不需要的前缀。 路由筛选是标准的网络做法，通常在多个网络中使用。

* 定义路由筛选器，并将其应用于 ExpressRoute 线路。 路由筛选器是一种新资源，可让你选择计划通过 Microsoft 对等互连使用的服务列表。 ExpressRoute 路由器仅发送属于路由筛选器中所标识服务的前缀列表。

在本教程中，你将了解如何执行以下操作：
> [!div class="checklist"]
> - 创建路由筛选器和筛选器规则。
> - 将路由筛选器关联到 ExpressRoute 线路。

### <a name="about-route-filters"></a><a name="about"></a>关于路由筛选器

在 ExpressRoute 线路上配置 Microsoft 对等互连时，Microsoft 边缘路由器会通过连接提供商的边缘路由器建立一对 BGP 会话。 不会将任何路由播发到网络。 若要能够将路由播发到网络，必须关联路由筛选器。

使用路由筛选器可标识要通过 ExpressRoute 线路的 Microsoft 对等互连使用的服务。 它实质上是所有 BGP 社区值的允许列表。 定义路由筛选器资源并将其附加到 ExpressRoute 线路后，映射到 BGP 社区值的所有前缀均会播发到网络。


> [!IMPORTANT]
> 在 2019 年 10 月 1 日之前配置的 ExpressRoute 线路的 Microsoft 对等互连会通过 Microsoft 对等互连播发所有服务前缀，即使未定义路由筛选器。 在 2019 年 10 月 1 日或之后配置的 ExpressRoute 线路的 Microsoft 对等互连的任何前缀只有在路由筛选器附加到线路之后才会播发。
> 
> 

## <a name="prerequisites"></a>先决条件

- 在开始配置之前，请查看[先决条件](expressroute-prerequisites.md)和[工作流](expressroute-workflows.md)。

- 必须具备预配了 Microsoft 对等互连的活动 ExpressRoute 线路。 可使用以下说明完成这些任务：
  - 继续下一步之前，请[创建 ExpressRoute 线路](expressroute-howto-circuit-arm.md)，并让连接提供商启用该线路。 ExpressRoute 线路必须处于已预配且已启用状态。
  - 如果直接管理 BGP 会话，请[创建 Microsoft 对等互连](expressroute-circuit-peerings.md)。 或者，让连接提供商为线路预配 Microsoft 对等互连。

-  必须创建并配置路由筛选器。
    - 标识要通过 Microsoft 对等互连使用的服务
    - 标识与服务关联的 BGP 团体值列表
    - 创建规则以允许前缀列表与 BGP 团体值相匹配

-  必须将路由筛选器附加到 ExpressRoute 线路。

## <a name="before-you-begin"></a>准备阶段

开始配置之前，请确保满足以下条件：

 - 在开始配置之前，请查看[先决条件](expressroute-prerequisites.md)和[工作流](expressroute-workflows.md)。

 - 必须有一个活动的 ExpressRoute 线路。 在继续下一步之前，请按说明 [创建 ExpressRoute 线路](expressroute-howto-circuit-arm.md) ，并通过连接提供商启用该线路。 ExpressRoute 线路必须处于已预配且已启用状态。

 - 必须有活动的 Microsoft 对等互连。 按照[创建和修改对等互连配置](expressroute-circuit-peerings.md)一文中的说明操作。


### <a name="working-with-azure-powershell"></a>使用 Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]


### <a name="log-in-to-your-azure-account"></a>登录 Azure 帐户

在开始此配置之前，必须登录到 Azure 帐户。 该 cmdlet 会提示提供 Azure 帐户的登录凭据。 登录后它会下载帐户设置，供 Azure PowerShell 使用。

使用提升的权限打开 PowerShell 控制台，并连接到帐户。 使用下面的示例来帮助连接。 

```azurepowershell
Connect-AzAccount
```

如果有多个 Azure 订阅，请查看该帐户的订阅。

```azurepowershell
Get-AzSubscription
```

指定要使用的订阅。

```azurepowershell
Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
```

## <a name="get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a> 获取前缀和 BGP 社区值的列表

1. 使用以下 cmdlet 获取与通过 Microsoft 对等互连可访问的服务相关联的 BGP 社区值和前缀列表：

```azurepowershell
Get-AzBgpServiceCommunity
```

1. 列出要在路由筛选器中使用的 BGP 团体值列表。

## <a name="create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>创建路由筛选器和筛选器规则

1 个路由筛选器只能有 1 个规则，并且规则类型必须是“允许”。 此规则可以有与之关联的 BGP 团体值列表。 命令 `az network route-filter create` 只创建路由筛选器资源。 创建资源后，必须创建规则并将其附加到路由筛选器对象。

1. 若要创建路由筛选器资源，请运行以下命令：

```azurepowershell
New-AzRouteFilter -Name "MyRouteFilter" -ResourceGroupName "MyResourceGroup" -Location "China East"
```

1. 若要创建路由筛选器规则，请运行以下命令：
 
```azurepowershell
$rule = New-AzRouteFilterRuleConfig -Name "Allow-EXO-D365" -Access Allow -RouteFilterRuleType Community -CommunityList 12076:5010,12076:5040
```

1. 运行以下命令将筛选器规则添加到路由筛选器：
 
```azurepowershell
$routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
$routefilter.Rules.Add($rule)
Set-AzRouteFilter -RouteFilter $routefilter
```

## <a name="attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>将路由筛选器附加到 ExpressRoute 线路

运行以下命令将路由筛选器附加到 ExpressRoute 线路，假设你只有 Microsoft 对等互连：

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
$ckt.Peerings[0].RouteFilter = $routefilter 
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="common-tasks"></a><a name="tasks"></a>常见任务

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>获取路由筛选器的属性

若要获取路由筛选器的属性，请使用以下步骤：

1. 运行以下命令来获取路由筛选器资源：

   ```azurepowershell
   $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
   ```
2. 通过运行以下命令获取路由筛选器资源的路由筛选器规则：

   ```azurepowershell
   $routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
   $rule = $routefilter.Rules[0]
   ```

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>更新路由筛选器的属性

如果路由筛选器已附加到线路，则 BGP 社区列表的更新会通过建立的 BGP 会话自动传播前缀播发更改。 可使用以下命令更新路由筛选器的 BGP 团体列表：

```azurepowershell
$routefilter = Get-AzRouteFilter -Name "RouteFilterName" -ResourceGroupName "ExpressRouteResourceGroupName"
$routefilter.rules[0].Communities = "12076:5030", "12076:5040"
Set-AzRouteFilter -RouteFilter $routefilter
```

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>从 ExpressRoute 线路分离路由筛选器

从 ExpressRoute 线路分离路由筛选器后，BGP 会话不会播发任何前缀。 可使用以下命令从 ExpressRoute 线路分离路由筛选器：
  
```azurepowershell
$ckt.Peerings[0].RouteFilter = $null
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="clean-up-resources"></a>清理资源

只有在路由筛选器未附加到任何线路时，才能将其删除。 尝试删除路由筛选器之前，请确保其未附加到任何线路。 可使用以下命令删除路由筛选器：

```azurepowershell
Remove-AzRouteFilter -Name "MyRouteFilter" -ResourceGroupName "MyResourceGroup"
```

## <a name="next-steps"></a>后续步骤

若要了解路由器配置示例，请参阅：

> [!div class="nextstepaction"]
> [用于设置和管理路由的路由器配置示例](expressroute-config-samples-routing.md)
