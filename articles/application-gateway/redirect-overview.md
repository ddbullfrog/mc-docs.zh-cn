---
title: Azure 应用程序网关的重定向概述
description: 了解 Azure 应用程序网关中的重定向功能，以将一个侦听器上收到的流量重定向到另一个侦听器或外部站点。
services: application-gateway
author: amsriva
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/29/2020
ms.author: v-junlch
ms.openlocfilehash: 22617612d033ecb688f0881e7f9352e8ed5778ad
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937432"
---
# <a name="application-gateway-redirect-overview"></a>应用程序网关重定向概述

可以使用应用程序网关来重定向流量。  它有一种泛型重定向机制，可以将一个侦听器上接收的流量重定向到另一个侦听器或外部站点。 这样可以简化应用程序配置、优化资源使用情况，并支持全局重定向和基于路径的重定向等新的重定向方案。

为确保应用程序及其用户之间的所有通信都通过加密路径进行，适用于许多 Web 应用的常见重定向方案是支持 HTTP 到 HTTPS 自动重定向。 过去用户曾使用创建专用的后端池等技术，其唯一目的在于将通过 HTTP 接收的请求重定向到 HTTPS。 由于应用程序网关提供重定向支持，因此你可以很容易地完成此操作，只需向路由规则添加一个新的重定向配置，然后将使用 HTTPS 协议的另一个侦听器指定为目标侦听器即可。

支持以下类型的重定向：

- 301 永久性重定向
- 302 已找到
- 303 参见其他
- 307 临时重定向

应用程序网关重定向支持具有以下功能：

-  **全局重定向**

   在网关上从一个侦听器重定向到另一个侦听器。 这样可实现站点上的 HTTP 到 HTTPS 重定向。
- **基于路径的重定向**

   这种类型的重定向只能在特定站点区域中进行 HTTP 到 HTTPS 重定向，例如 /cart/* 表示的购物车区域。
- **重定向到外部站点**

![关系图显示了用户和应用网关以及两者之间的连接，包括未锁定的 HTTP 红色箭头，不允许的 301 直接红色箭头以及已锁定的 HTTPS 绿色箭头。](./media/redirect-overview/redirect.png)

进行此更改后，客户需要创建新的重定向配置对象，以指定重定向需要指向的目标侦听器或外部站点。 配置元素还支持一些选项，通过这些选项可以将 URI 路径和查询字符串追加到重定向的 URL。 也可选择重定向的类型。 创建此重定向配置后，会通过新规则将其附加到源侦听器。 使用基本规则时，重定向配置与源侦听器相关联，并且是全局重定向。 使用基于路径的规则时，将在 URL 路径映射中定义重定向配置。 因此，它仅适用于站点的特定路径区域。

### <a name="next-steps"></a>后续步骤

[在应用程序网关上配置 URL 重定向](tutorial-url-redirect-powershell.md)

