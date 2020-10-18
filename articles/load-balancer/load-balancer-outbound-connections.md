---
title: 出站连接
titleSuffix: Azure Load Balancer
description: 本文介绍 Azure 如何使 VM 与公共 Internet 服务通信。
services: load-balancer
documentationcenter: na
author: WenJason
ms.service: load-balancer
ms.custom: contperfq1
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 09/24/2020
ms.date: 10/19/2020
ms.author: v-jay
ms.openlocfilehash: 5a05b10de51ff2c0cad674bd9b893f1894a247d9
ms.sourcegitcommit: 57511ab990fbb26305a76beee48f0c223963f7ca
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/12/2020
ms.locfileid: "91943504"
---
# <a name="outbound-connections"></a>出站连接

Azure 负载均衡器可通过各种机制提供出站连接。 本文介绍了各种方案以及如何管理它们。 

## <a name="outbound-connections-scenario-overview"></a><a name="scenarios"></a>出站连接方案概述

在这些方案中使用的术语。 有关详细信息，请参阅[术语](#terms)：

* [源网络地址转换 (SNAT)](#snat)
* [端口伪装 (PAT)](#pat)
* 传输控制协议 (TCP)
* 用户数据报协议 (UDP)
* 网络地址转换
* Internet 控制消息协议
* 封装安全协议

### <a name="scenarios"></a>方案

* [方案 1](#scenario1) - 具有公共 IP 的虚拟机。
* [方案 2](#scenario2) - 没有公共 IP 的虚拟机。
* [方案 3](#scenario3) - 没有公共 IP 且没有标准负载均衡器的虚拟机。

### <a name="scenario-1---virtual-machine-with-public-ip"></a><a name="scenario1"></a>方案 1 - 具有公共 IP 的虚拟机

| 关联 | 方法 | IP 协议 |
| ---------- | ------ | ------------ |
| 公共负载均衡器或独立 | [SNAT](#snat) </br> 不使用[端口伪装](#pat)。 | TCP </br> UDP </br> ICMP </br> ESP |

#### <a name="description"></a>描述

Azure 将分配给实例 NIC 的 IP 配置的公共 IP 用于所有出站流。 此实例具有所有可用的临时端口。 VM 是否负载均衡无关紧要。 此方案优先于其他方案。 

分配到 VM 的公共 IP 属于 1 对 1 关系（而不是 1 对多关系），并实现为无状态的 1 对 1 NAT。

### <a name="scenario-2---virtual-machine-without-public-ip"></a><a name="scenario2"></a>方案 2 - 没有公共 IP 的虚拟机

| 关联 | 方法 | IP 协议 |
| ------------ | ------ | ------------ |
| 公共负载均衡器 | 将负载均衡器前端用于带有[端口伪装 (PAT)](#pat) 功能的 [SNAT](#snat)。| TCP </br> UDP |

#### <a name="description"></a>描述

负载均衡器资源使用负载均衡器规则进行配置。 此规则用于在公共 IP 前端与后端池之间创建链接。 

如果没有完成此规则配置，则行为将如方案 3 所述。 

不需要使用包含侦听器的规则即可成功进行运行状况探测。

当 VM 创建出站流时，Azure 会将源 IP 地址转换为公共负载均衡器前端的公共 IP 地址。 此转换通过 [SNAT](#snat) 完成。 

负载均衡器的前端公共 IP 地址的临时端口用于区分源自 VM 的各个流。 创建出站流后，SNAT 动态使用[预先分配的临时端口](#preallocatedports)。 

在此情况下，用于 SNAT 的临时端口被称为 SNAT 端口。 SNAT 端口是预先分配的，如[默认 SNAT 端口分配表](#snatporttable)所述。

### <a name="scenario-3---virtual-machine-without-public-ip-and-without-standard-load-balancer"></a><a name="scenario3"></a> 方案 3 - 没有公共 IP 且没有标准负载均衡器的虚拟机

| 关联 | 方法 | IP 协议 |
| ------------ | ------ | ------------ |
|无 </br> 基本负载均衡器 | 带有[端口伪装 (PAT)](#pat) 功能的 [SNAT](#snat)| TCP </br> UDP | 

#### <a name="description"></a>描述

当 VM 创建出站流时，Azure 将此出站流的源 IP 地址转换为公共源 IP 地址。 此公共 IP 地址不可配置且无法保留。 针对订阅的公共 IP 资源限制进行计数时，不会计入此地址。 

如果你重新部署以下项，系统会释放此公共 IP 地址并请求新的公共 IP： 

* 虚拟机
* 可用性集
* 虚拟机规模集  

不要将此方案用于向允许列表添加 IP。 请使用方案 1 或 2，你可以在其中显式声明出站行为。 [SNAT](#snat) 端口是预先分配的，如[默认 SNAT 端口分配表](#snatporttable)所述。



## <a name="port-allocation-algorithm"></a><a name="preallocatedports"></a>端口分配算法

Azure 使用一个算法来确定可用的已预先分配的 [SNAT](#snat) 端口数。 此算法将端口数基于后端池的大小。 

对于与负载均衡器关联的每个公共 IP 地址，有 64,000 个端口可用作 [SNAT](#snat) 端口。 相同数量的 [SNAT](#snat) 端口会预先分配给 UDP 和 TCP。 将根据 IP 传输协议独立地使用这些端口。 

[SNAT](#snat) 端口使用情况会有所不同，具体取决于流是 UDP 还是 TCP。 

端口会动态消耗，直至达到预先分配的限制。  当流关闭或出现空闲超时时，会释放这些端口。

有关空闲超时的详细信息，请参阅[排查 Azure 负载均衡器中的出站连接问题](../load-balancer/troubleshoot-outbound-connection.md#idletimeout) 

仅当需要使流保持唯一时，才使用端口。

### <a name="dynamic-snat-ports-preallocation"></a><a name="snatporttable"></a> 动态 SNAT 端口的预先分配

下表显示了针对后端池大小的层级的 [SNAT](#snat) 端口预分配情况：

| 池大小（VM 实例） | 每个 IP 配置的预先分配 SNAT 端口 |
| --- | --- |
| 1-50 | 1,024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1,000 | 32 |

更改后端池大小可能会影响建立的某些流：

- 后端池大小增加会触发将所在层级转换成下一个层级。 在转换成下一个层级的过程中，一半的预先分配的 [SNAT](#snat) 端口会被回收。 

- 与回收的 [SNAT](#snat) 端口关联的流会超时并关闭。 这些流必须重建。 如果尝试新流，则只要预先分配的端口可用，则该流就能立即成功。

- 如果后端池减小并转换为更低层级，可用的 [SNAT](#snat) 端口数会增多。 现有的给定 [SNAT](#snat) 端口及其相应的流不会受影响。

## <a name="outbound-rules"></a><a name="outboundrules"></a>出站规则

 可以使用出站规则配置公共[标准负载均衡器](load-balancer-standard-overview.md)的出站网络地址转换。  

> [!NOTE]
> **Azure 虚拟网络 NAT** 可以为虚拟网络中的虚拟机提供出站连接。  有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

你可以根据自己的需求，以完全声明性的方式控制出站连接，以缩放和优化此功能。 本部分扩展了上面所述的方案 2。

![负载均衡器出站规则](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

使用出站规则，你可以使用负载均衡器从头开始定义出站 NAT。 你还可以缩放和优化现有出站 NAT 的行为。

使用出站规则可以控制：

- 哪些虚拟机应转换为哪些公共 IP 地址。
- 应如何提供出站 [SNAT](#snat) 端口。
- 要为哪些协议提供出站转换。
- 用于出站连接空闲超时的持续时间（4-120 分钟）。
- 是否要在空闲超时时发送 TCP 重置
- 带有单个规则的 TCP 和 UDP 传输协议

### <a name="outbound-rule-definition"></a>出站规则定义

出站规则遵循用户熟悉的与负载均衡和入站 NAT 规则相同的语法：**前端** + **参数** + **后端池**。 出站规则为后端池识别的、要转换为前端的所有虚拟机配置出站 NAT。   参数针对出站 NAT 算法提供更精细的控制。

### <a name="scale-outbound-nat-with-multiple-ip-addresses"></a><a name="scale"></a>使用多个 IP 地址缩放出站 NAT

前端提供的每个附加 IP 地址可提供额外的 64,000 个临时端口，供负载均衡器用作 SNAT 端口。 

使用多个 IP 地址来规划大规模方案。 使用出站规则来缓解 [SNAT 耗尽](troubleshoot-outbound-connection.md#snatexhaust)的情况。 

你还可以直接在出站规则中使用[公共 IP 前缀](/load-balancer/load-balancer-outbound-connections#outboundrules)。 

公共 IP 前缀增强了部署的缩放。 可以将前缀添加到源自 Azure 资源的流的允许列表中。 可以在负载均衡器中配置引用公共 IP 地址前缀所需的前端 IP 配置。  

负载均衡器可控制公共 IP 前缀。 出站规则会自动使用公共 IP 前缀中包含的所有公共 IP 地址来建立出站连接。 

公共 IP 前缀范围内的每个 IP 地址可提供额外的 64,000 个临时端口，供负载均衡器用作 SNAT 端口。

### <a name="outbound-flow-idle-timeout-and-tcp-reset"></a><a name="idletimeout"></a> 出站流空闲超时和 TCP 重置

出站规则提供一个配置参数用于控制出站流空闲超时，并使该超时符合应用程序的需求。 出站空闲超时默认为 4 分钟。 有关详细信息，请参阅[配置空闲超时](load-balancer-tcp-idle-timeout.md#tcp-idle-timeout)。 

负载均衡器的默认行为是在达到了出站空闲超时时以静默方式丢弃流。 `enableTCPReset` 参数可以让应用程序的行为和控制更具可预测性。 此参数指示在发生出站空闲超时时是否要发送双向 TCP 重置 (TCP RST)。 

查看[在空闲超时时 TCP 重置](/load-balancer/load-balancer-tcp-reset)，了解详细信息，包括区域可用性。

### <a name="preventing-outbound-connectivity"></a><a name="preventoutbound"></a>阻止出站连接

负载均衡规则提供出站 NAT 的自动编程。 某些方案受益于或者要求通过负载均衡规则禁用出站 NAT 的自动编程。 通过该规则进行禁用可以控制或优化行为。  

可通过两种方式使用此参数：

1. 禁止将入站 IP 地址用于出站 SNAT。 在负载均衡规则中禁用出站 SNAT。
  
2. 对同时用于入站和出站连接的 IP 地址的出站 [SNAT](#snat) 参数进行优化。 必须禁用自动出站 NAT 才能让出站规则掌管控制权。 若要更改也用于入站连接的某个地址的 SNAT 端口分配，则必须将 `disableOutboundSnat` 参数设置为 true。 

如果尝试重新定义用于入站连接的 IP 地址，则配置出站规则的操作会失败。  请先禁用负载均衡规则的出站 NAT。

>[!IMPORTANT]
> 如果将此参数设置为 true，但没有任何出站规则来定义出站连接，则虚拟机将不会建立出站连接。  VM或应用程序的某些操作可能依赖于公网连接。 请务必了解方案的依赖关系，并考虑此项更改造成的影响。

有时，让 VM 创建出站流是不合需要的。 可能需要对哪些目标接收出站流或哪些目标启动入站流进行管理。 使用[网络安全组](../virtual-network/security-overview.md)可管理 VM 访问的目标。 使用 NSG 可对哪些公共目标启动入站流进行管理。

将 NSG 应用于负载均衡的 VM 时，需要注意[服务标记](../virtual-network/security-overview.md#service-tags)和[默认安全规则](../virtual-network/security-overview.md#default-security-rules)。 请确保 VM 可以接收来自 Azure 负载均衡器的运行状况探测请求。

如果 NSG 阻止来自 AZURE_LOADBALANCER 默认标记的运行状况探测请求，那么 VM 的运行状况探测程序将失败，并且 VM 被标记为停机。 负载均衡器停止向此 VM 发送新流。

## <a name="scenarios-with-outbound-rules"></a>具有出站规则的方案

### <a name="outbound-rules-scenarios"></a>出站规则方案

* [方案 1](#scenario1out) - 将出站连接配置为源自一组特定的公共 IP 或前缀。
* [方案 2](#scenario2out) - 修改 [SNAT](#snat) 端口分配。
* [方案 3](#scenario3out) - 仅启用出站连接。
* [方案 4](#scenario4out) - 仅对 VM 使用出站 NAT（无入站 NAT）。
* [方案 5](#scenario5out) - 内部标准负载均衡器的出站 NAT。
* [方案 6](#scenario6out) - 使用公共标准负载均衡器为出站 NAT 启用 TCP 和 UDP 协议。

### <a name="scenario-1"></a><a name="scenario1out"></a>方案 1

| 方案 |
| -------- |
| 将出站连接配置为源自一组特定的公共 IP 或前缀|

#### <a name="details"></a>详细信息

使用此方案将出站连接调整成源自一组公共 IP 地址。 根据来源向允许列表或拒绝列表添加公共 IP 或前缀。

此公共 IP 或前缀可与负载均衡规则使用的相同。 

若要使用与负载均衡规则使用的公共 IP 或前缀不同的公共 IP 或前缀，请执行以下操作：  

1. 创建公共 IP 前缀或公共 IP 地址。
2. 创建公共标准负载均衡器 
3. 创建一个前端，用于引用所要使用的公共 IP 前缀或公共 IP 地址。 
4. 重复使用某个后端池或创建一个后端池，并将 VM 放入公共负载均衡器的后端池
5. 在公共负载均衡器上配置出站规则，以使用前端为这些 VM 启用出站 NAT。 如果不希望将负载均衡规则用于出站连接，则在负载均衡规则中禁用出站 SNAT。

### <a name="scenario-2"></a><a name="scenario2out"></a>方案 2

| 方案 |
| -------- |
| 修改 [SNAT](#snat) 端口分配 |

#### <a name="details"></a>详细信息

可以使用出站规则[基于后端池大小优化自动 SNAT 端口分配](load-balancer-outbound-connections.md#preallocatedports)。 

如果遇到 SNAT 耗尽的情况，请增加给定的 [SNAT](#snat) 端口数（默认值为 1024）。 

每个公共 IP 地址最多提供 64,000 个临时端口。 后端池中的 VM 数决定了分配给每个 VM 的端口数。 后端池中的一个 VM 最多可以访问 64,000 个端口。 对于两个 VM 的情况，可以使用出站规则为每个 VM 最多指定 32,000 个 [SNAT](#snat) 端口 (2x 32,000 = 64,000)。 

可以使用出站规则来优化默认情况下给定的 SNAT 端口。 你指定的端口数可以多于或少于默认 [SNAT](#snat) 端口分配提供的端口数。 出站规则的前端中的每个公共 IP 地址最多提供 64,000 个可用作 [SNAT](#snat) 端口的临时端口。  

负载均衡器以 8 的倍数提供 [SNAT](#snat) 端口。 如果提供的值不能被 8 整除，则会拒绝配置操作。 

如果你尝试指定的 [SNAT](#snat) 端口数超出了系统所能提供的端口数（具体取决于公共 IP 地址数），系统会拒绝该配置操作。

如果你为每个 VM 指定 10,000 个端口，而后端池中 7 个 VM 共享 1 个公共 IP 地址，系统会拒绝该配置。 7 乘以 10,000 超出了 64,000 个端口的限制。 将更多的公共 IP 地址添加到出站规则的前端即可实现该方案。 

将端口数指定为 0 即可恢复到[默认端口分配](load-balancer-outbound-connections.md#preallocatedports)。 前 50 个 VM 实例会获得 1024 个端口，而 51-100 个 VM 实例会获得 512 个端口，以此类推，直到最大实例数。  有关默认 SNAT 端口分配的详细信息，请参阅[上文](#snatporttable)。

### <a name="scenario-3"></a><a name="scenario3out"></a>方案 3

| 方案 |
| -------- |
| 仅启用出站连接 |

#### <a name="details"></a>详细信息

可以使用公共标准负载均衡器为一组 VM 提供出站 NAT。 在此方案中，可以单独使用出站规则，而无需其他任何规则。

> [!NOTE]
> **Azure 虚拟网络 NAT** 可以为虚拟机提供出站连接，无需使用负载均衡器。  有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

### <a name="scenario-4"></a><a name="scenario4out"></a>方案 4

| 方案 |
| -------- |
| 仅对 VM 使用出站 NAT（无入站连接） |

> [!NOTE]
> **Azure 虚拟网络 NAT** 可以为虚拟机提供出站连接，无需使用负载均衡器。  有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

#### <a name="details"></a>详细信息

对于此方案，请执行以下操作：

1. 创建公共 IP 或前缀。
2. 创建公共标准负载均衡器。 
3. 创建一个与专用于出站的公共 IP 或前缀关联的前端。
4. 为 VM 创建后端池。
5. 将 VM 置于后端池。
6. 配置启用出站 NAT 所需的出站规则。

使用前缀或公共 IP 来缩放 [SNAT](#snat) 端口。 将出站连接的源添加到允许列表或拒绝列表。

### <a name="scenario-5"></a><a name="scenario5out"></a>方案 5

| 方案 |
| -------- |
| 内部标准负载均衡器的出站 NAT |

> [!NOTE]
> **Azure 虚拟网络 NAT** 可以利用内部标准负载均衡器为虚拟机提供出站连接。  有关详细信息，请参阅[什么是 Azure 虚拟网络 NAT？](../virtual-network/nat-overview.md)。

#### <a name="details"></a>详细信息

在显式声明出站连接之前，出站连接对于内部标准负载均衡器来说不可用。 

有关详细信息，请参阅[仅出站的负载均衡器配置](/load-balancer/egress-only)。


### <a name="scenario-6"></a><a name="scenario6out"></a>方案 6

| 方案 |
| -------- |
| 使用公共标准负载均衡器为出站 NAT 启用 TCP 和 UDP 协议 |

#### <a name="details"></a>详细信息

使用公共标准负载均衡器时，提供的自动出站 NAT 与负载均衡规则的传输协议相匹配。 

1. 在负载均衡规则中禁用出站 [SNAT](#snat)。 
2. 在同一个负载均衡器上配置出站规则。
3. 重复使用 VM 已用的后端池。 
4. 指定“协议”：“所有”作为出站规则的一部分。 

只使用入站 NAT 规则时，不会提供出站 NAT。 

1. 将 VM 放入后端池。
2. 使用公共 IP 地址或公共 IP 前缀定义一个或多个前端 IP 配置 
3. 在同一个负载均衡器上配置出站规则。 
4. 指定“协议”：“所有”作为出站规则的一部分

## <a name="terminology"></a><a name="terms"></a> 术语

### <a name="source-network-address-translation"></a><a name="snat"></a>源网络地址转换

| 适用的协议 |
|------------------------|
| TCP </br> UDP          |

#### <a name="details"></a>详细信息

Azure 中的部署可与 Azure 外部的公共 IP 地址空间中的终结点进行通信。

当实例开始向某个具有公共 IP 地址的目标发送出站流量时，Azure 会将资源的专用 IP 地址动态映射到公共 IP 地址。 

创建此映射后，此出站发起流的返回流量会抵达发起流的专用 IP 地址。 

Azure 使用**源网络地址转换 (SNAT)** 来执行此功能。

### <a name="port-masquerading-snat-pat"></a><a name="pat"></a>端口伪装 SNAT (PAT)

| 适用的协议 |
|------------------------|
| TCP </br> UDP          |

#### <a name="details"></a>详细信息

当专用 IP 位于单个公共 IP 地址之后时，Azure 会使用**端口地址转换 (PAT)** 来隐藏专用 IP 地址。 

临时端口用于 PAT，是基于池大小[预先分配](#preallocatedports)的。 

当公共负载均衡器与没有公共 IP 的 VM 相关联时，系统会重写每个出站连接源。 

出站连接源会从虚拟网络专用 IP 地址重写为负载均衡器的前端公共 IP 地址。 

在公共 IP 地址空间中，下面的流的五元组必须唯一：

* 源 IP 地址
* Source Port
* IP 传输协议
* 目标 IP 地址
* Destination Port 

端口伪装 SNAT 可与 TCP 或 UDP 协议配合使用。 重写专用源 IP 地址后会使用 SNAT 端口，因为多个流源自单个公共 IP 地址。 端口伪装 SNAT 算法分别以不同方式为 UDP 与 TCP 指定 SNAT 端口。

### <a name="snat-ports-tcp"></a>SNAT 端口 (TCP)

| 适用的协议 |
|------------------------|
| TCP |

#### <a name="details"></a>详细信息

SNAT 端口是可用于公共 IP 源地址的临时端口。 每个到单个目标 IP 地址的流和端口使用一个 SNAT 端口。 

对于到相同的目标 IP 地址、端口和协议的多个 TCP 流，每个 TCP 流使用一个 SNAT 端口。 

这样使用可以确保流在从同一公共 IP 地址发出及到达以下目标时是独一无二的：

* 同一目标 IP 地址
* 端口
* 协议 

多个流共享单个 SNAT 端口。 

目标 IP 地址、端口和协议使流保持唯一，无需使用其他源端口来区分公共 IP 地址空间中的流。


### <a name="snat-ports-udp"></a>SNAT 端口 (UDP)

| 适用的协议 |
|------------------------|
| UDP |

#### <a name="details"></a>详细信息

UDP SNAT 端口由与 TCP SNAT 端口不同的算法管理。  负载均衡器对 UDP 使用名为“端口受限锥形 NAT”的算法。

不管目标 IP 地址和端口如何，每个流只使用一个 SNAT 端口。


### <a name="exhaustion"></a><a name="exhaustion"></a>耗尽

| 适用的协议 |
|------------------------|
| 空值 |

#### <a name="details"></a>详细信息

如果 SNAT 端口资源已经耗尽，那么在现有流释放 SNAT 端口之前出站流会失败。 当流关闭时，负载均衡器会回收 SNAT 端口。

负载均衡器使用 4 分钟[空闲超时](../load-balancer/troubleshoot-outbound-connection.md#idletimeout)回收 SNAT 端口。

由于所用算法的差异，UDP SNAT 端口的耗尽速度通常比 TCP SNAT 端口快得多。 请根据这种差异来进行设计和规模测试。

### <a name="snat-port-release-behavior-tcp"></a>SNAT 端口释放行为 (TCP)

| 适用的协议 |
|------------------------|
| TCP |

#### <a name="details"></a>详细信息

当服务器或客户端发送 FINACK 时，系统会在 240 秒后释放 SNAT 端口。 如果出现 RST，则会在 15 秒后释放 SNAT 端口。 如果已空闲超时，则会释放端口。

### <a name="snat-port-release-behavior-udp"></a>SNAT 端口释放行为 (UDP)

| 适用的协议 |
|------------------------|
| TCP |

#### <a name="details"></a>详细信息

如果已达到空闲超时，则会释放端口。

### <a name="snat-port-reuse"></a>SNAT 端口重用

| 适用的协议 |
|------------------------|
| TCP </br> UDP |

#### <a name="details"></a>详细信息

释放某个端口以后，即可重复使用该端口。 SNAT 端口是一个适用于给定方案的从低到高的序列，第一个可用 SNAT 端口用于新的连接。

## <a name="limitations"></a>限制

- 每个前端 IP 地址的最大可用临时端口数为 64,000。
- 可配置的出站空闲超时范围为 4 到 120 分钟（240 到 7200 秒）。
- 负载均衡器不支持将 ICMP 用于出站 NAT。
- 出站规则只能应用于 NIC 的主 IP 配置。  不能为 VM 或 NVA 的辅助 IP 创建出站规则。 支持多个 NIC。
- 使用内部标准负载均衡器时，不带虚拟网络和其他 Azure 平台服务的 Web 辅助角色可能是可访问的。 该可访问性是由于 VNet 出现之前的服务和其他平台服务的运行方式带来的副作用。 请勿依赖于此副作用，因为相应的服务本身或底层平台可能会在没有通知的情况下进行更改。 请始终假定你需要明确创建出站连接（如果在仅使用内部标准负载均衡器时需要这样做）。 本文中所述的方案 3 不可用。

## <a name="next-steps"></a>后续步骤

如果通过 Azure 负载均衡器进行出站连接时遇到问题，请参阅[出站连接故障排除指南](../load-balancer/troubleshoot-outbound-connection.md)。

- 详细了解[标准负载均衡器](load-balancer-standard-overview.md)。
- 请参阅 [Azure 负载均衡器常见问题解答](load-balancer-faqs.md)。
- 详细了解标准公共负载均衡器的[出站规则](load-balancer-outbound-rules-overview.md)。

