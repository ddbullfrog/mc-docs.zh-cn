---
title: Azure 防火墙 DNS 设置（预览版）
description: 可以使用 DNS 服务器和 DNS 代理设置来配置 Azure 防火墙。
services: firewall
ms.service: firewall
ms.topic: how-to
origin.date: 06/30/2020
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes
ms.testdate: 08/03/2020
ms.author: v-yeche
ms.openlocfilehash: 25a017bd5cde6e915cd42c799a632473662baaa2
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327289"
---
<!--Verify Successfully-->
# <a name="azure-firewall-dns-settings-preview"></a>Azure 防火墙 DNS 设置（预览版）

> [!IMPORTANT]
> Azure 防火墙 DNS 设置目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅[适用于 Azure 预览版的补充使用条款](https://www.azure.cn/support/legal/subscription-agreement/)。

可以为 Azure 防火墙配置自定义 DNS 服务器并启用 DNS 代理。 可以在部署防火墙时配置这些设置，或者稍后从“DNS 设置”页进行配置。

## <a name="dns-servers"></a>DNS 服务器

DNS 服务器维护域名并将它解析为 IP 地址。 默认情况下，Azure 防火墙将 Azure DNS 用于名称解析。 通过“DNS 服务器”设置，你可以配置自己的 DNS 服务器来用于 Azure 防火墙名称解析。 可以配置一台或多台服务器。

### <a name="configure-custom-dns-servers-preview"></a>配置自定义 DNS 服务器（预览版）

1. 在 Azure 防火墙的“设置”下，选择“DNS 设置” 。
2. 在“DNS 服务器”下，可以键入或添加之前在虚拟网络中指定的现有 DNS 服务器。
3. 选择“保存” 。
4. 防火墙现在将 DNS 流量定向到指定的 DNS 服务器以进行名称解析。

:::image type="content" source="media/dns-settings/dns-servers.png" alt-text="DNS 服务器":::

## <a name="dns-proxy-preview"></a>DNS 代理（预览）

可以对 Azure 防火墙进行配置来充当 DNS 代理。 DNS 代理充当从客户端虚拟机到 DNS 服务器的 DNS 请求的中介。 如果配置自定义 DNS 服务器，应启用 DNS 代理以避免 DNS 解析不匹配，并在网络规则中启用 FQDN 筛选。

如果未启用 DNS 代理，来自客户端的 DNS 请求可能会在不同的时间传到 DNS 服务器，或者返回与防火墙不同的响应。 DNS 代理将 Azure 防火墙放置在客户端请求的路径中以避免不一致。

配置 DNS 代理需要三个步骤：
1. 在 Azure 防火墙 DNS 设置中启用 DNS 代理。
2. （可选）配置自定义 DNS 服务器或使用提供的默认设置。
3. 最后，必须在虚拟网络 DNS 服务器设置中将 Azure 防火墙的专用 IP 地址配置为自定义 DNS 地址。 这可确保 DNS 流量定向到 Azure 防火墙。

### <a name="configure-dns-proxy-preview"></a>配置 DNS 代理（预览）

若要配置 DNS 代理，必须将你的虚拟网络 DNS 服务器设置配置为使用防火墙专用 IP 地址。 然后，在 Azure 防火墙策略“DNS 设置”中启用 DNS 代理。

#### <a name="configure-virtual-network-dns-servers"></a>配置虚拟网络 DNS 服务器

1. 选择 DNS 流量将通过 Azure 防火墙路由的虚拟网络。
2. 在“设置”下，选择“DNS 服务器”。 
3. 在“DNS 服务器”下，选择“自定义” 。
4. 输入防火墙的专用 IP 地址。
5. 选择“保存” 。
6. 重启已连接到虚拟网络的 VM，以便为其分配新的 DNS 服务器设置。 VM 在重启之前，将继续使用其当前 DNS 设置。

#### <a name="enable-dns-proxy-preview"></a>启用 DNS 代理（预览）

1. 选择 Azure 防火墙。
2. 在“设置”下，选择“DNS 设置” 。
3. 默认情况下，“DNS 代理”已禁用。 启用后，防火墙会在端口 53 上进行侦听，并将 DNS 请求转发到配置的 DNS 服务器。
4. 查看“DNS 服务器”配置以确保设置适用于你的环境。
5. 选择“保存”。

:::image type="content" source="media/dns-settings/dns-proxy.png" alt-text="DNS 代理":::

## <a name="next-steps"></a>后续步骤

[网络规则中的 FQDN 筛选](fqdn-filtering-network-rules.md)

<!-- Update_Description: update meta properties, wording update, update link -->