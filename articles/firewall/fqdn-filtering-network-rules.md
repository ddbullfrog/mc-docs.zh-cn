---
title: 网络规则中的 Azure 防火墙 FQDN 筛选（预览版）
description: 如何使用网络规则中的 Azure 防火墙 FQDN 筛选
services: firewall
ms.service: firewall
ms.topic: article
origin.date: 08/25/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 6d0f153b93ecbf8e31272362c631ad7def6f9a9b
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246797"
---
<!--Verified successfully-->
# <a name="use-fqdn-filtering-in-network-rules-preview"></a>在网络规则中使用 FQDN 筛选（预览版）

> [!IMPORTANT]
> 网络规则中的 FQDN 筛选目前为公共预览版。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅[适用于 Azure 预览版的补充使用条款](https://www.azure.cn/support/legal/subscription-agreement/)。

完全限定的域名 (FQDN) 表示主机或 IP 地址的域名。 你可以基于 Azure 防火墙和防火墙策略中的 DNS 解析在网络规则中使用 FQDN。 此功能允许你筛选采用任何 TCP/UDP 协议（包括 NTP、SSH、RDP 等）的出站流量。 若要在网络规则中使用 FQDN，你必须启用 DNS 代理。 有关详细信息，请参阅 [Azure 防火墙 DNS 设置（预览版）](dns-settings.md)。

> [!NOTE]
> 按照设计，FQDN 筛选不支持通配符。

## <a name="how-it-works"></a>工作原理

定义你的组织需要的 DNS 服务器（Azure DNS 或你自己的自定义 DNS）后，Azure 防火墙将基于所选 DNS 服务器将 FQDN 转换为 IP 地址。 将针对应用程序和网络规则处理进行此转换。

在应用程序规则中使用域名与在网络规则中使用域名有何区别？ 

- 应用程序规则中针对 HTTP/S 和 MSSQL 的 FQDN 筛选基于应用程序级透明代理和 SNI 标头。 因此，它可以区分解析为同一 IP 地址的两个 FQDN。 对于网络规则中的 FQDN 筛选，情况并非如此。 如果可能，请始终使用应用程序规则。
- 在应用程序规则中，可以使用 HTTP/S 和 MSSQL 作为所选协议。 在网络规则中，可以将任何 TCP/UDP 协议与目标 FQDN 结合使用。

## <a name="next-steps"></a>后续步骤

[Azure 防火墙 DNS 设置](dns-settings.md)

<!-- Update_Description: update meta properties, wording update, update link -->