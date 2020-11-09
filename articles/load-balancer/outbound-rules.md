---
title: 使用 Azure 负载均衡器配置出站规则
description: 本文介绍如何使用 Azure 负载均衡器配置出站规则来控制 Internet 流量的流出量。
services: load-balancer
author: WenJason
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperfq1
origin.date: 10/13/2020
ms.date: 11/02/2020
ms.author: v-jay
ms.openlocfilehash: 27d5b5139ec6cb1a1ec10e228b8fda27216d92e2
ms.sourcegitcommit: 1f933e4790b799ceedc685a0cea80b1f1c595f3d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629780"
---
# <a name="outbound-rules-azure-load-balancer"></a><a name="outboundrules"></a>使用 Azure 负载均衡器配置出站规则

通过出站规则，你可以配置公共标准负载均衡器的出站 SNAT（源网络地址转换）。 使用此配置，你可以将负载均衡器的公共 IP 用作代理。

此配置可实现：

* IP 伪装
* 简化允许列表。
* 减少用于部署的公共 IP 资源的数量。

使用出站规则，你可以完全声明性地控制出站 Internet 连接。 通过出站规则，你可以根据特定需要微调和调整此功能。 

仅当后端 VM 没有实例级公共 IP 地址 (ILPIP) 时，才会遵循出站规则。

![负载均衡器出站规则](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

可以使用出站规则显式定义出站 SNAT 行为。

使用出站规则可以控制：

* **哪些虚拟机应转换为哪些公共 IP 地址。**
     * 有两个规则，后端池 A 使用 IP 地址 A 和 B，后端池 B 使用 IP 地址 C 和 D。
* **应如何提供出站 SNAT 端口。**
     * 后端池 B 是唯一建立出站连接的池，将所有 SNAT 端口提供给后端池 B，而无 SNAT 端口提供给后端池 A。
* **要为哪些协议提供出站转换。**
     * 后端池 B 需要 UDP 端口才能建立出站连接。 后端池 A 需要 TCP。 把 TCP 端口提供给 A，把 UDP 端口提供给 B。
* **用于出站连接空闲超时的持续时间（4-120 分钟）。**
     * 如果有长时间运行的带有 keepalives 的连接，请为长时间运行的连接保留空闲端口，空闲时间最长可达 120 分钟。 假设放弃过时连接，并在 4 分钟内为新连接释放端口 
* **是否要在空闲超时时发送 TCP 重置。**
     * 当空闲连接超时时，我们是否会向客户端和服务器发送 TCP RST，以便它们知道已放弃流？

## <a name="outbound-rule-definition"></a>出站规则定义

出站规则遵循用户熟悉的与负载均衡和入站 NAT 规则相同的语法： **前端** + **参数** + **后端池** 。 

出站规则为后端池识别的、要转换为前端的所有虚拟机配置出站 NAT。   

参数针对出站 NAT 算法提供更精细的控制。

## <a name="scale-outbound-nat-with-multiple-ip-addresses"></a><a name="scale"></a>使用多个 IP 地址缩放出站 NAT

前端提供的每个附加 IP 地址可提供额外的 64,000 个临时端口，供负载均衡器用作 SNAT 端口。 

使用多个 IP 地址来规划大规模方案。 使用出站规则来缓解 [SNAT 耗尽](troubleshoot-outbound-connection.md#snatexhaust)的情况。 

你还可以直接在出站规则中使用[公共 IP 前缀](/load-balancer/load-balancer-outbound-connections#outboundrules)。 

公共 IP 前缀增强了部署的缩放。 可以将前缀添加到源自 Azure 资源的流的允许列表中。 可以在负载均衡器中配置引用公共 IP 地址前缀所需的前端 IP 配置。  

负载均衡器可控制公共 IP 前缀。 出站规则会自动使用公共 IP 前缀中包含的所有公共 IP 地址来建立出站连接。 

公共 IP 前缀范围内的每个 IP 地址可提供额外的 64,000 个临时端口，供负载均衡器用作 SNAT 端口。

## <a name="outbound-flow-idle-timeout-and-tcp-reset"></a><a name="idletimeout"></a> 出站流空闲超时和 TCP 重置

出站规则提供一个配置参数用于控制出站流空闲超时，并使该超时符合应用程序的需求。 出站空闲超时默认为 4 分钟。 有关详细信息，请参阅[配置空闲超时](load-balancer-tcp-idle-timeout.md)。 

负载均衡器的默认行为是在达到了出站空闲超时时以静默方式丢弃流。 `enableTCPReset` 参数可以让应用程序的行为和控制更具可预测性。 此参数指示在发生出站空闲超时时是否要发送双向 TCP 重置 (TCP RST)。 

查看[在空闲超时时 TCP 重置](/load-balancer/load-balancer-tcp-reset)，了解详细信息，包括区域可用性。

## <a name="securing-and-controlling-outbound-connectivity-explicitly"></a><a name="preventoutbound"></a>显式保护和控制出站连接

负载均衡规则提供出站 NAT 的自动编程。 某些方案受益于或者要求通过负载均衡规则禁用出站 NAT 的自动编程。 通过该规则进行禁用可以控制或优化行为。  

可通过两种方式使用此参数：

1. 禁止将入站 IP 地址用于出站 SNAT。 在负载均衡规则中禁用出站 SNAT。
  
2. 对同时用于入站和出站连接的 IP 地址的出站 **SNAT** 参数进行优化。 必须禁用自动出站 NAT 才能让出站规则掌管控制权。 若要更改也用于入站连接的某个地址的 SNAT 端口分配，则必须将 `disableOutboundSnat` 参数设置为 true。 

如果尝试重新定义用于入站连接的 IP 地址，则配置出站规则的操作会失败。  请先禁用负载均衡规则的出站 NAT。

>[!IMPORTANT]
> 如果将此参数设置为 true，但没有任何出站规则来定义出站连接，则虚拟机将不会建立出站连接。  VM或应用程序的某些操作可能依赖于公网连接。 请务必了解方案的依赖关系，并考虑此项更改造成的影响。

有时，让 VM 创建出站流是不合需要的。 可能需要对哪些目标接收出站流或哪些目标启动入站流进行管理。 使用[网络安全组](../virtual-network/security-overview.md)可管理 VM 访问的目标。 使用 NSG 可对哪些公共目标启动入站流进行管理。

将 NSG 应用于负载均衡的 VM 时，需要注意[服务标记](../virtual-network/security-overview.md#service-tags)和[默认安全规则](../virtual-network/security-overview.md#default-security-rules)。 

请确保 VM 可以接收来自 Azure 负载均衡器的运行状况探测请求。

如果 NSG 阻止来自 AZURE_LOADBALANCER 默认标记的运行状况探测请求，那么 VM 的运行状况探测程序将失败，并且 VM 被标记为不可用。 负载均衡器停止向此 VM 发送新流。

## <a name="limitations"></a>限制

- 每个前端 IP 地址的最大可用临时端口数为 64,000。
- 可配置的出站空闲超时范围为 4 到 120 分钟（240 到 7200 秒）。
- 负载均衡器不支持将 ICMP 用于出站 NAT。
- 出站规则只能应用于 NIC 的主 IP 配置。  不能为 VM 或 NVA 的辅助 IP 创建出站规则。 支持多个 NIC。
- 可用性集中的所有虚拟机都必须添加到后端池以进行出站连接。 
- 虚拟机规模集中的所有虚拟机都必须添加到后端池以进行出站连接。

## <a name="next-steps"></a>后续步骤

- 详细了解 [Azure 标准负载均衡器](load-balancer-overview.md)
- 请参阅 [Azure 负载均衡器常见问题解答](load-balancer-faqs.md)

