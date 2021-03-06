---
title: 关于升级线路带宽 | Azure ExpressRoute
description: 本文介绍升级 ExpressRoute 线路带宽的最佳做法
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: cherylmc
ms.openlocfilehash: 66c5b7eac180ecb65253ea9a353f7ac0d909bf50
ms.sourcegitcommit: 9d9795f8a5b50cd5ccc19d3a2773817836446912
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2020
ms.locfileid: "88228970"
---
# <a name="about-upgrading-expressroute-circuit-bandwidth"></a>关于升级 ExpressRoute 线路带宽

可以通过 ExpressRoute 与 Microsoft 的全球网络建立专用连接。 可以通过 ExpressRoute 合作伙伴的网络或与 Microsoft Enterprise Edge (MSEE) 设备的直接连接来快速建立连接。 配置并测试物理连接后，可以通过创建 ExpressRoute 线路并配置对等互连来启用第 2 层和第 3 层连接。

## <a name="upgrade-circuit-bandwidth"></a><a name="upgrade"></a>升级线路带宽

在升级线路带宽时，ExpressRoute Direct 或 ExpressRoute 合作伙伴需要有[足够的可用带宽](#considerations)才能成功升级。

如果有容量，可以使用以下方法升级线路：

* [Azure 门户](expressroute-howto-circuit-portal-resource-manager.md#modify)
* [PowerShell](expressroute-howto-circuit-arm.md#modify)
* [Azure CLI](howto-circuit-cli.md#modify)

## <a name="capacity-considerations"></a><a name="considerations"></a>容量注意事项

### <a name="insufficient-expressroute-partner-bandwidth"></a><a name="bandwidth"></a>ExpressRoute 合作伙伴带宽不足

如果 ExpressRoute 合作伙伴没有足够的容量，你需要创建一个新线路，并根据所需带宽对其进行配置。 为了保持连接性，在预配完新建的线路、配置完对等互连并（针对专用对等互连）预配完与 ExpressRoute 虚拟网关的连接对象之前，请不要删除旧线路。

如果 ExpressRoute 合作伙伴没有足够的可用容量，你需要在所需的对等互连位置请求额外的容量。 预配新容量后，可按照相关文章的[升级线路带宽](#upgrade)部分中包含的步骤来创建新线路、配置连接并删除旧线路。




## <a name="next-steps"></a>后续步骤

* [创建和修改线路](expressroute-howto-circuit-portal-resource-manager.md)
* [创建和修改对等配置](expressroute-howto-routing-portal-resource-manager.md)
* [将虚拟网络链接到 ExpressRoute 线路](expressroute-howto-linkvnet-portal-resource-manager.md)
