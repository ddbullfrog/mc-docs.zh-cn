---
title: 出站代理 Azure 负载均衡器
description: 介绍如何将 Azure 负载均衡器用作出站 Internet 连接的代理
services: load-balancer
author: WenJason
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperfq1
origin.date: 10/13/2020
ms.date: 11/02/2020
ms.author: v-jay
ms.openlocfilehash: befba1d45920005a9769d95486d7432c64d13b01
ms.sourcegitcommit: 1f933e4790b799ceedc685a0cea80b1f1c595f3d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92628209"
---
# <a name="outbound-proxy-azure-load-balancer"></a>出站代理 Azure 负载均衡器

Azure 负载均衡器可以用作出站 Internet 连接的代理。 负载均衡器可为后端实例提供出站连接。 

此配置使用源网络地址转换 (SNAT)。 SNAT 将后端的 IP 地址重写为负载均衡器的公共 IP 地址。 

SNAT 启用后端实例的 IP 伪装。 此伪装可以防止外部源直接访问后端实例。 

在后端实例之间共享 IP 地址可降低静态公共 IP 的成本，并支持简化带有来自已知公共 IP 的流量的 IP 允许列表等场景。 

## <a name="sharing-ports-across-resources"></a><a name ="snat"></a> 跨资源共享端口

如果负载均衡器的后端资源没有实例级别公共 IP (ILPIP) 地址，则它们会通过公共负载均衡器的前端 IP 建立出站连接。

端口用于生成用于维护不同流的唯一标识符。 Internet 使用五元组来提供这种区别。

5 元组包含：

* 目标 IP
* 目标端口
* 源 IP
* 源端口和协议可以提供这种区别。

如果一个端口用于入站连接，它将有一个用于该端口上入站连接请求的侦听器，且它不能用于出站连接。 

若要建立出站连接，必须使用临时端口为目标提供一个端口，在该端口上进行通信并维护不同的通信流。 

每个 IP 地址具有 65,535 个端口。 将前 1024 个端口保留为系统端口。 每个端口都可以用于 TCP 和 UDP 的入站或出站连接。 

在剩余的端口中，Azure 提供 64,000 个端口作为临时端口。 当将 IP 地址添加为前端 IP 配置时，这些临时端口可用于 SNAT。

通过出站规则，可以将这些 SNAT 端口分发到后端实例，使它们能够共享负载均衡器的公共 IP，实现出站连接。

主机上每个后端实例的网络都将对出站连接中的数据包进行 SNAT。 主机将源 IP 重写为一个公共 IP。 主机将每个出站数据包的源端口重写为一个 SNAT 端口。

## <a name="exhausting-ports"></a><a name="scenarios"></a> 大量消耗端口

每个连接到同一目标 IP 和目标端口的连接都将使用 SNAT 端口。 此连接维护从后端实例或从客户端到服务器的不同流量  。 这个过程为服务器提供了一个不同的端口来处理流量。 如果没有此过程，客户端计算机将无法知道数据包属于哪个流。

假设有多个浏览器将访问 https://www.microsoft.com ，即：

* 目标 IP = 23.53.254.142
* 目标端口 = 443
* 协议 = TCP

如果返回流量的目标端口（用于建立连接的 SNAT 端口）都相同，客户端将无法将一个查询结果与另一个查询结果分开。

出站连接可能会突发。 后端实例可能无法分配到足够的端口。 如果未启用连接重用，则会增加 SNAT 端口耗尽的风险 。

当端口耗尽时，与目标 IP 的新出站连接将失败。 当端口变为可用时，连接将成功。 当来自 IP 地址的 64,000 个端口在许多后端实例上分散分布时，就会发生这种耗尽。 有关缓解 SNAT 端口耗尽的指导，请参阅[故障排除指南](/load-balancer/troubleshoot-outbound-connection)。  

对于 TCP 连接，负载均衡器将为每个目标 IP 和端口使用一个 SNAT 端口。 这种多用途允许使用相同 SNAT 端口建立与相同目标 IP 的多个连接。 如果连接不是指向不同的目标端口，那么这种多用途是有限的。

对于 UDP 连接，负载均衡器使用端口受限的 cone NAT 算法，无论目标端口是什么，每个目标 IP 都会消耗一个 SNAT 端口。 

可以在无限数量的连接中重复使用端口。 仅当目标 IP 或端口不同时，才可重复使用端口。

## <a name="port-allocation"></a><a name="preallocatedports"></a> 端口分配

作为负载均衡器的前端 IP 分配的每个公共 IP 都会为其后端池成员分配 64,000 个 SNAT 端口。 这些端口无法与后端池成员共享。 一系列的 SNAT 端口只能由单个后端实例使用，这样才可确保正确路由返回包。 

建议使用显式出站规则来配置 SNAT 端口分配。 此规则将使每个后端实例可用于出站连接的 SNAT 端口数最多。 

如果选择通过负载均衡规则自动分配出站 SNAT，则分配表将定义端口分配。

下表<a name="snatporttable"></a>显示了针对后端池大小的层级的 SNAT 端口预分配情况：

| 池大小（VM 实例） | 每个 IP 配置的预先分配 SNAT 端口 |
| --- | --- |
| 1-50 | 1,024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1,000 | 32 | 

>[!NOTE]
> 如果你有一个最大大小为 6 的后端池，且定义了一个显式出站规则，则每个实例可以有 64,000/10 = 6,400 个端口。 根据上表，如果选择自动分配，则每个实例只有 1,024 个端口。

## <a name="outbound-rules-and-virtual-network-nat"></a><a name="outboundrules"></a> 出站规则和虚拟网络 NAT

Azure 负载均衡器出站规则和虚拟网络 NAT 是用于虚拟网络流出量的选项。

有关出站规则的详细信息，请参阅[出站规则](outbound-rules.md)。

有关 Azure 虚拟网络 NAT 的详细信息，请参阅[什么是 Azure 虚拟网络 NAT](../virtual-network/nat-overview.md)。

## <a name="constraints"></a>约束

* 接收或发送 TCP RST 后，将在 15 秒后释放端口
* 接收或发送 FINACK 后，将在 240 秒后释放端口
* 如果连接处于闲置状态且没有发送新的数据包，则将在 4 - 120 分钟后释放端口。
  * 可以通过出站规则配置此阈值。
* 每个 IP 地址提供 64,000 个端口，这些端口可用于 SNAT。
* 每个端口都可以用于到目标 IP 地址的 TCP 和 UDP 连接
  * 无论目标端口是否唯一，都需要 UDP SNAT 端口。 对于每个到目标 IP 的 UDP 连接，将使用一个 UDP SNAT 端口。
  * 如果目标端口不同，则可以将一个 TCP SNAT 端口用于到同一目标 IP 的多个连接。
* 当后端实例用完给定的 SNAT 端口时，会发生 SNAT 耗尽。 负载均衡器仍然可以有未使用的 SNAT 端口。 如果后端实例的已用 SNAT 端口超过其给定的 SNAT 端口，它将无法建立新的出站连接。

## <a name="next-steps"></a>后续步骤

* [SNAT 耗尽造成的出站连接失败故障排除](/load-balancer/troubleshoot-outbound-connection)
* [查看 SNAT 指标](/load-balancer/load-balancer-standard-diagnostics#how-do-i-check-my-snat-port-usage-and-allocation)并熟悉筛选、拆分和查看它们的正确方法。

