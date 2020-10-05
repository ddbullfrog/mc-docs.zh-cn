---
title: Azure 流量管理器 - 常见问题解答
description: 本文提供有关流量管理器的常见问题解答
services: traffic-manager
documentationcenter: ''
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 02/26/2019
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: yes
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.openlocfilehash: ae8428d6e1215d1be864d9225bea9be9d59f7666
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395155"
---
# <a name="traffic-manager-frequently-asked-questions-faq"></a>流量管理器常见问题解答 (FAQ)

## <a name="traffic-manager-basics"></a>流量管理器基础知识

### <a name="what-ip-address-does-traffic-manager-use"></a>流量管理器使用什么 IP 地址？

如[流量管理器工作原理](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理器在 DNS 级别工作。 它会发送 DNS 响应，将客户端定向到相应的服务终结点。 然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。

因此，流量管理器不提供供客户端连接的终结点或 IP 地址。 如果想要为服务使用静态 IP 地址，必须在服务而不是流量管理器中配置该地址。

### <a name="what-types-of-traffic-can-be-routed-using-traffic-manager"></a>可以使用流量管理器路由什么类型的流量？
如[流量管理器工作原理](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理器终结点可以是任何面向 Internet 的 Azure 内部或外部托管的服务。 因此，流量管理器可以将源自公共 Internet 的流量路由到一组也面向 Internet 的终结点。 如果终结点位于专用网络内部（例如 [Azure 负载均衡器](../load-balancer/components.md#frontend-ip-configurations)的内部版本），或用户从此类内部网络发出 DNS 请求，则无法使用流量管理器来路由此流量。

### <a name="does-traffic-manager-support-sticky-sessions"></a>流量管理器是否支持“粘滞”会话？

如[流量管理器工作原理](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理器在 DNS 级别工作。 它使用 DNS 响应将客户端引导到相应的服务终结点。 客户端直接连接到服务终结点，而不是通过流量管理器连接。 因此，流量管理器看不到客户端与服务器之间的 HTTP 流量。

此外，流量管理器收到的 DNS 查询的源 IP 地址属于递归 DNS 服务而不是客户端。 因此，流量管理器无法跟踪单个客户端，也无法实现“粘滞”会话。 这种限制在所有基于 DNS 的流量管理系统中很常见，并不是流量管理器特有的限制。

### <a name="why-am-i-seeing-an-http-error-when-using-traffic-manager"></a>使用流量管理器时为何出现 HTTP 错误？

如[流量管理器工作原理](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理器在 DNS 级别工作。 它使用 DNS 响应将客户端引导到相应的服务终结点。 然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。 流量管理器看不到客户端与服务器之间的 HTTP 流量。 因此，出现的任何 HTTP 错误必定来自应用程序。 要使客户端连接到应用程序，必须完成所有 DNS 解析步骤。 这包括流量管理器对应用程序流量流所做的任何交互。

因此，进一步的调查应着重于应用程序。

问题的最常见原因源自客户端浏览器发送的 HTTP 主机标头。 请确保将应用程序配置为接受所要使用的域名的正确主机标头。 对于使用 Azure 应用服务的终结点，请参阅[使用流量管理器为 Azure 应用服务中的 Web 应用配置自定义域名](../app-service/configure-domain-traffic-manager.md)。

### <a name="what-is-the-performance-impact-of-using-traffic-manager"></a>使用流量管理器对性能有什么影响？

如[流量管理器工作原理](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理器在 DNS 级别工作。 由于客户端直接连接到服务终结点，因此在使用流量管理器时，一旦建立连接就没有性能影响。

由于流量管理器在 DNS 级别与应用程序集成，因此需要将额外的 DNS 查找插入 DNS 解析链中。 流量管理器对 DNS 解析时间的影响微乎其微。 流量管理器使用全局性的名称服务器网络，并使用任播网络来确保始终将 DNS 查询路由到最靠近的可用名称服务器。 此外，对 DNS 响应进行缓存意味着，因使用流量管理器而导致的额外的 DNS 延迟仅出现在部分会话中。

<!--Not Available on  [anycast](https://en.wikipedia.org/wiki/Anycast)-->

“性能”方法将流量路由到最靠近的可用终结点。 最终结果是，与此方法相关的整体性能影响会减到最小。 通过减小与终结点之间的网络延迟，应该可以抵消 DNS 延迟增大造成的影响。

### <a name="what-application-protocols-can-i-use-with-traffic-manager"></a>流量管理器允许使用什么应用程序协议？

如[流量管理器工作原理](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理器在 DNS 级别工作。 完成 DNS 查找以后，客户端会直接连接到应用程序终结点，不通过流量管理器进行连接。 因此，连接可以使用任何应用程序协议。 如果选择 TCP 作为监视协议，则无需使用任何应用程序协议即可执行流量管理器的终结点运行状况监视。 如果选择使用应用程序协议验证运行状态，则终结点需要能够响应 HTTP 或 HTTPS GET 请求。

### <a name="can-i-use-traffic-manager-with-a-naked-domain-name"></a>是否可以对“裸”域名使用流量管理器？

是的。 若要了解如何为域名顶点创建别名记录以引用 Azure 流量管理器配置文件，请参阅[使用流量管理器配置支持顶点域名的别名记录](https://docs.azure.cn/dns/tutorial-alias-tm)。

### <a name="does-traffic-manager-consider-the-client-subnet-address-when-handling-dns-queries"></a>处理 DNS 查询时流量管理器是否会考虑客户端子网地址？ 

会，除了其收到的 DNS 查询的源 IP 地址（通常是 DNS 解析器的 IP 地址）以外，在执行地理、性能和子网路由方法的查找时，流量管理器还会考虑客户端子网地址（如果它通过代表最终用户发起请求的解析器包含在查询中）。  
具体而言，[RFC 7871 - DNS 查询中的客户端子网](https://tools.ietf.org/html/rfc7871)可提供 [DNS (EDNS0) 的扩展机制](https://tools.ietf.org/html/rfc2671)，可通过客户端子网地址从支持该机制的解析器传递该机制。

### <a name="what-is-dns-ttl-and-how-does-it-impact-my-users"></a>什么是 DNS TTL，它如何影响我的用户？

当某个 DNS 查询进入流量管理器时，它会在响应中设置一个名为生存时间 (TTL) 的值。 此值的单位为秒，向下游的 DNS 解析程序指明要将此响应缓存多长时间。 尽管无法保证 DNS 解析程序会缓存此结果，但如果缓存结果的话，解析程序就能通过缓存响应所有后续查询，而无需转到流量管理器 DNS 服务器。 这会对响应造成如下所述的影响：

- 更高的 TTL 可以减少进入流量管理器 DNS 服务器的查询数目，从而又可以降低客户的成本，因为根据查询数目提供服务是要收费的。
- 更高的 TTL 可能会减少执行 DNS 查找所需的时间。
- 更高的 TTL 还意味着，数据不会反映流量管理器通过其探测代理获取到的最新运行状况信息。

### <a name="how-high-or-low-can-i-set-the-ttl-for-traffic-manager-responses"></a>可将流量管理器响应的 TTL 设置为多高或多低？

在配置文件级别，DNS TTL 最低可设置为 0 秒，最高可设置为 2,147,483,647 秒（符合 [RFC-1035](https://www.ietf.org/rfc/rfc1035.txt ) 的最大范围）。 TTL 为 0 表示下游 DNS 解析程序不会缓存查询响应，所有查询预期会进入流量管理器 DNS 服务器供解析。

### <a name="how-can-i-understand-the-volume-of-queries-coming-to-my-profile"></a>如何了解传入到我的配置文件的查询数量？ 

流量管理器提供的指标之一是由配置文件响应的查询数。 可以在配置文件聚合级别获取此信息，也可以进一步对其进行拆分来查看返回了特定终结点的查询数量。 此外，还可以设置警报，以便在查询响应数量越过了设置的条件时向你发出通知。 有关更多详细信息，请参阅[流量管理器指标和警报](traffic-manager-metrics-alerts.md)。

## <a name="traffic-manager-geographic-traffic-routing-method"></a>流量管理器的“地理”流量路由方法

### <a name="what-are-some-use-cases-where-geographic-routing-is-useful"></a>可以在哪些情况下使用地理路由？

只要 Azure 客户需要根据地理区域辨识其用户，即可使用地理路由类型。 例如，通过使用地理流量路由方法可为特定区域的用户提供不同于其他区域的用户体验。 又比如，根据本地数据自主性的规定，只能由同一区域的终结点为用户提供数据。

### <a name="how-do-i-decide-if-i-should-use-performance-routing-method-or-geographic-routing-method"></a>如何决定我应当使用性能路由方法还是应当使用地理路由方法？

这两种常用路由方法的主要区别为，在性能路由方法中，主要目标是将流量发送到可以为调用方提供最低延迟的终结点，但是，在地理路由中，主要目标是针对调用方实施地理围栏，以便可以特意将它们路由到特定终结点。 因为地理接近与延迟更低之间存在关联，所以这两种路由方法可能会存在重叠，但是并非始终会重叠。 如果另一地理位置中的终结点可以为调用方提供更好的延迟体验，在这种情况下，性能路由会将用户发送到该终结点，但地理路由始终将用户发送到你为其地理区域映射的终结点。 在这种情况下，地理路由将特意执行你为其配置的要执行的确切操作，性能优化不是一个考虑因素。 
>[!NOTE]
>在某些场景中，你可能同时需要性能和地理路由功能，对于这些场景，嵌套式配置文件可能是不错的选择。 

### <a name="what-are-the-regions-that-are-supported-by-traffic-manager-for-geographic-routing"></a>进行地理路由时，流量管理器支持哪些区域？

可在[此处](traffic-manager-geographic-regions.md)查找流量管理器使用的国家/地区层次结构。 更改会在此页进行更新，不过，也可以通过 [Azure 流量管理器 REST API](https://docs.microsoft.com/rest/api/trafficmanager/) 以编程方式检索相同的信息。 

### <a name="how-does-traffic-manager-determine-where-a-user-is-querying-from"></a>流量管理器如何确定用户从何处进行查询？

流量管理器会查看查询的源 IP（很可能是本地 DNS 解析程序在代表用户执行查询），并使用内部 IP 通过区域映射的方式确定位置。 该映射会随时更新，以反映 Internet 中的变化。 

### <a name="is-it-guaranteed-that-traffic-manager-can-correctly-determine-the-exact-geographic-location-of-the-user-in-every-case"></a>是否可以保证流量管理器在每种情况下都可正确确定用户的确切地理位置？

不可以；流量管理器无法保证我们根据 DNS 查询的源 IP 地址推断出的地理区域始终对应用户的位置，原因如下：

- 首先，如之前常见问题解答中所述，我们所见的源 IP 地址是代表用户执行查询的 DNS 解析器的 IP 地址。 虽然 DNS 解析器的地理位置很好地反映了用户的地理位置，但根据 DNS 解析器服务的足迹和用户选择使用的特定 DNS 解析器服务而有所不同。 举例来说，位于马来西亚的客户可能在其设备设置中指定使用一种 DNS 解析器服务，但可能会选取该服务在新加坡的 DNS 服务器来处理此用户/设备的查询解析。 这种情况下，流量管理器仅会显示对应于新加坡位置的解析程序的 IP 地址。 另请参阅本页面上之前有关客户端子网地址支持的常见问题解答。

- 其次，流量管理器使用内部映射来执行 IP 地址到地理区域的转换。 尽管此映射会经过不断验证和更新来提高其准确性和阐释 Internet 的演变，但是我们的信息仍有可能不能确切反应所有 IP 地址的地理位置。

### <a name="does-an-endpoint-need-to-be-physically-located-in-the-same-region-as-the-one-it-is-configured-with-for-geographic-routing"></a>是否需将终结点与进行地理路由时用来进行配置的终结点置于同一区域？

否。终结点的位置不会限制可以向其映射哪些区域。 

### <a name="can-i-assign-geographic-regions-to-endpoints-in-a-profile-that-is-not-configured-to-do-geographic-routing"></a>是否可以将地理区域分配给某个配置文件中的终结点，而该配置文件尚未进行地理路由所需的配置？

可以；如果某个配置文件的路由方法不是地理路由，则可使用 [Azure 流量管理器 REST API](https://docs.microsoft.com/rest/api/trafficmanager/) 将地理区域分配给该配置文件中的终结点。 如果是非地理路由类型的配置文件，则会忽略该配置。 如果在以后将此类配置文件更改为地理路由类型，流量管理器可以使用相应的映射。

### <a name="why-am-i-getting-an-error-when-i-try-to-change-the-routing-method-of-an-existing-profile-to-geographic"></a>尝试将现有配置文件的路由方法更改为地理路由时，为何会出现错误？

使用地理路由时，配置文件中的所有终结点都至少需要将一个区域映射到其中。 要将现有配置文件转换为地理路由类型，首先需使用 [Azure 流量管理器 REST API](https://docs.microsoft.com/rest/api/trafficmanager/) 将地理区域关联到所有终结点，此后再将路由类型更改为地理路由。 如果使用门户，请先删除终结点，将配置文件的路由方法更改为地理路由，然后再添加终结点及其地理区域映射。

### <a name="why-is-it-strongly-recommended-that-customers-create-nested-profiles-instead-of-endpoints-under-a-profile-with-geographic-routing-enabled"></a>为何强烈建议客户创建嵌套式配置文件，而不是将终结点直接置于启用了地理路由的配置文件中？

如果使用地理路由方法，则只能将一个区域分配给配置文件中的一个终结点。 如果该终结点不是带有子配置文件的嵌套类型，则当该终结点不正常时，流量管理器仍会继续向其发送流量，因为不发送流量也不会造成任何改善。 流量管理器不会故障转移到其他终结点，即使所分配的区域是分配给不正常终结点的区域的“父”区域。 这样做是为了确保流量管理器遵守客户在其配置文件中设置的地理边界。 为了确保在某个终结点不正常时能够故障转移到其他终结点，建议为地理区域分配包含多个终结点（而不是单个终结点）的嵌套式配置文件。 这样一来，如果嵌套式子配置文件中的某个终结点故障，则可将流量故障转移到同一嵌套式子配置文件中的其他终结点。

### <a name="are-there-any-restrictions-on-the-api-version-that-supports-this-routing-type"></a>对于支持此路由类型的 API 版本，是否存在任何限制？

是的。仅 API 2017-03-01 及更高版本支持地理路由类型。 不能使用旧的 API 版本创建地理路由类型的配置文件或向终结点分配地理区域。 如果使用旧的 API 版本从 Azure 订阅检索配置文件，则不会返回任何地理路由类型的配置文件。 另外，在使用旧的 API 版本时，如果返回的配置文件的终结点进行了地理区域分配，则不会显示该地理区域分配。

## <a name="traffic-manager-subnet-traffic-routing-method"></a>流量管理器子网流量路由方法

### <a name="what-are-some-use-cases-where-subnet-routing-is-useful"></a>可以在哪些情况下使用子网路由？

针对按 DNS 请求 IP 地址的源 IP 标识的特定用户组，子网路由可以为其提供不同的体验。 如果用户从公司总部连接到网站，则会显示不同的内容。 另一种情况是限制来自某些 ISP 的用户，使其只能访问仅支持 IPv4 连接的终结点（针对在使用 IPv6 时性能不佳的 ISP）。
使用子网路由方法的另一个原因是其能与嵌套配置文件集中的其他配置文件配合使用。 例如，如果要使用地理路由方法对用户进行地域隔离，但想要对特定 ISP 使用不同的路由方法，则可以将具有子网路由方法的配置文件作为父级配置文件，并覆盖该 ISP 以使用特定的子级配置文件，并为其他所有人提供标准的地理配置文件。

### <a name="how-does-traffic-manager-know-the-ip-address-of-the-end-user"></a>流量管理器如何获取最终用户的 IP 地址？

最终用户设备通常使用 DNS 解析器来代表他们执行 DNS 查找。 流量管理器将此类解析器的传出 IP 视为源 IP。 此外，子网路由方法还会查看是否存在随请求一起传递的 EDNS0 扩展客户端子网 (ECS) 信息。 如果存在 ECS 信息，则将该地址用于确定路由。 如果缺少 ECS 信息，则将查询的源 IP 用于路由。

### <a name="how-can-i-specify-ip-addresses-when-using-subnet-routing"></a>使用子网路由时如何指定 IP 地址？

可以通过两种方式指定与终结点相关联的 IP 地址。 首先，可以使用带有起始和结束地址的四分点十进制八位字节表示法来指定范围（例如，1.2.3.4-5.6.7.8 或 3.4.5.6-3.4.5.6）。 其次，可以使用 CIDR 表示法来指定范围（例如 1.2.3.0/24）。 可以指定多个范围，并且可以在范围集中使用这两种表示法类型。 存在一些限制。

- 由于每个 IP 必须映射到单个终结点，因此地址范围不能重叠
- 起始地址不能超过结束地址
- 在 CIDR 表示法的情况下，“/”之前的 IP 地址应该是该范围的起始地址（例如 1.2.3.0/24 有效但 1.2.3.4.4/24 无效）

### <a name="how-can-i-specify-a-fallback-endpoint-when-using-subnet-routing"></a>使用子网路由时如何指定回退终结点？

在具有子网路由的配置文件中，如果没有子网映射到某个终结点，则所有与其他终结点不匹配的请求都将定向到此终结点。 强烈建议在配置文件中配置这样一个回退终结点，因为如果请求进入并且未映射到任何终结点或者如果它映射到终结点但该终结点存在问题，流量管理器将返回 NXDOMAIN 响应。

### <a name="what-happens-if-an-endpoint-is-disabled-in-a-subnet-routing-type-profile"></a>如果在子网路由类型配置文件中禁用终结点会发生什么？

在具有子网路由的配置文件中，如果终结点已禁用，则流量管理器会像该终结点及其子网映射不存在一样运行。 如果收到与其 IP 地址映射匹配的查询且终结点已禁用，则流量管理器将返回回退终结点（没有映射的终结点），或者如果不存在回退终结点，将返回 NXDOMAIN 响应。

## <a name="traffic-manager-multivalue-traffic-routing-method"></a>流量管理器的多值流量路由方法

### <a name="what-are-some-use-cases-where-multivalue-routing-is-useful"></a>可以在哪些情况下使用多值路由？

多值路由在单个查询响应中返回多个运行正常的终结点。 此方法的主要优势在于，如果某个终结点运行不正常，则客户端有多个选项可以进行重试，而无需另外进行 DNS 调用（这样做可能会从上游缓存返回相同的值）。 这适用于希望最大程度地减少停机时间、对可用性敏感的应用程序。
另一种可以使用多值路由方法的情况是：终结点“双归属”于 IPv4 和 IPv6 地址，且你想在调用方启动与该终结点的连接时为其提供这两个选项。

### <a name="how-many-endpoints-are-returned-when-multivalue-routing-is-used"></a>使用多值路由会返回多少终结点？

可以指定返回的最大终结点数，当收到查询时，多值路由将返回不超过该指定数量的正常运行的终结点。 此配置的最大值为 10。

### <a name="will-i-get-the-same-set-of-endpoints-when-multivalue-routing-is-used"></a>使用多值路由时，会收到一组相同的终结点吗？

我们无法保证在每个查询中都会返回相同的一组终结点。 此结果也受终结点的运行状况影响，在响应中不包括某些终结点时，说明它们可能存在问题

<!-- Not Available ## Real User Measurements-->
<!-- Not Available ## Traffic View-->

## <a name="traffic-manager-endpoints"></a>流量管理器终结点

### <a name="can-i-use-traffic-manager-with-endpoints-from-multiple-subscriptions"></a>能否将流量管理器用于多个订阅的终结点？

不能对 Azure Web 应用使用多个订阅中的终结点。 Web 应用要求其所用的任何自定义域名只能在单个订阅中使用。 无法对多个订阅中的 Web 应用使用同一个域名。

对于其他终结点类型，可在多个订阅中结合使用流量管理器和终结点。 在 Resource Manager 中，只要配置流量管理器配置文件的人员具有终结点的读取访问权限，任何订阅的终结点就都可添加到流量管理器中。 可使用 [Azure 基于角色的访问控制 (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) 授予这些权限。 可使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) 或 [Azure CLI](https://docs.azure.cn/cli/network/traffic-manager/endpoint#az-network-traffic-manager-endpoint-create) 添加其他订阅中的终结点。

### <a name="can-i-use-traffic-manager-with-cloud-service-staging-slots"></a>能否将流量管理器用于云服务的“过渡”槽？

是的。 可以在流量管理器中将云服务的“过渡”槽配置为“外部”终结点。 运行状况检查仍按 Azure 终结点费率计费。

### <a name="does-traffic-manager-support-ipv6-endpoints"></a>流量管理器是否支持 IPv6 终结点？

流量管理器当前不提供 IPv6 可寻址的名称服务器。 但是，IPv6 客户端仍可使用流量管理器连接到 IPv6 终结点。 客户端不会直接向流量管理器发出 DNS 请求， 而是使用递归 DNS 服务。 仅使用 IPv6 的客户端通过 IPv6 向递归 DNS 服务发送请求。 然后，该递归服务应该能够使用 IPv4 联系流量管理器名称服务器。

流量管理器使用终结点的 DNS 名称或 IP 地址做出响应。 要支持 IPv6 终结点，有两个选择。 可将终结点添加为与 AAAA 记录相关联的 DNS 名称，流量管理器将对该终结点进行运行状况检查，并会在查询响应中将其作为 CNAME 记录类型返回。 还可以使用 IPv6 地址直接添加该终结点，流量管理器将在查询响应中返回 AAAA 类型记录。

### <a name="can-i-use-traffic-manager-with-more-than-one-web-app-in-the-same-region"></a>流量管理器是否可用于同一区域中的多个 Web 应用？

通常情况下，流量管理器用于将流量引导到部署在不同区域中的应用程序。 不过，流量管理器也可用于一个应用程序在同一区域中存在多个部署的情形。 流量管理器 Azure 终结点不允许将同一 Azure 区域中的多个 Web 应用终结点添加到同一流量管理器配置文件。

### <a name="how-do-i-move-my-traffic-manager-profiles-azure-endpoints-to-a-different-resource-group-or-subscription"></a>如何将流量管理器配置文件的 Azure 终结点移到其他资源组或订阅？

与流量管理器配置文件关联的 Azure 终结点使用其资源 ID 进行跟踪。 如果 Azure 资源作为终结点（例如公共 IP、经典云服务、WebApp 或以嵌套方式使用的另一流量管理器配置文件）使用，当它移到其他资源组或订阅时，其资源 ID 会发生更改。 对于这种情况，目前必须删除流量管理器配置文件，方法是先删除这些终结点，然后再将其添加回配置文件。

## <a name="traffic-manager-endpoint-monitoring"></a>流量管理器终结点监视

### <a name="is-traffic-manager-resilient-to-azure-region-failures"></a>流量管理器能否灵活应对 Azure 区域故障？

流量管理器是在 Azure 中提供高可用性应用程序的关键组件。
若要提供高可用性，流量管理器必须具有超高的可用性，并且能够灵活应对区域故障。

在设计上，流量管理器组件能够灵活应对任何 Azure 区域的全面故障。 这样的应对能力见于所有流量管理器组件：DNS 名称服务器、API、存储层、终结点监视服务。

在整个 Azure 区域发生服务中断的情况下（这种情况很少见），流量管理器仍可继续正常运行。 在多个 Azure 区域中部署的应用程序可以依赖流量管理器将流量定向到这些区域中的可用应用程序实例。

### <a name="how-does-the-choice-of-resource-group-location-affect-traffic-manager"></a>选择资源组位置会如何影响流量管理器？

流量管理器属于单一的全局性服务， 而不是区域性服务。 选择资源组位置对部署在该资源组中的流量管理器配置文件没有影响。

Azure Resource Manager 要求所有资源组指定一个位置，这决定了部署在该资源组中的资源的默认位置。 创建流量管理器配置文件时，会在资源组中创建该配置文件。 所有流量管理器配置文件使用“**全局**”作为位置，覆盖资源组的默认值。

### <a name="how-do-i-determine-the-current-health-of-each-endpoint"></a>如何确定每个终结点的当前运行状况？

除了总体配置文件，每个终结点的当前监视状态也显示在 Azure 门户中。 此信息也可通过流量监视器 [REST API](https://msdn.microsoft.com/library/azure/mt163667.aspx)、[PowerShell cmdlet](https://docs.microsoft.com/powershell/module/az.trafficmanager) 和[跨平台 Azure CLI](../cli-install-nodejs.md) 获取。

可以使用 Azure Monitor 来跟踪终结点的运行状况，并查看其可视表示形式。 有关使用 Azure Monitor 的详细信息，请参阅 [Azure 监视文档](/monitoring-and-diagnostics/monitoring-overview-metrics)。

### <a name="can-i-monitor-https-endpoints"></a>能否监视 HTTPS 终结点？

是的。 流量管理器支持通过 HTTPS 进行探测。 在监视配置中将 **HTTPS** 配置为协议。

流量管理器无法提供任何证书验证，包括：

* 不验证服务器端证书
* 不验证 SNI 服务器端证书
* 不支持客户端证书

### <a name="do-i-use-an-ip-address-or-a-dns-name-when-adding-an-endpoint"></a>添加终结点时是否使用 IP 地址或 DNS 名称？

流量管理器支持使用三种方式添加终结点：DNS 名称、IPv4 地址和 IPv6 地址。 如果将终结点添加为 IPv4 或 IPv6 地址，查询响应将分别为记录类型 A 或 AAAA。 如果终结点已添加为 DNS 名称，则查询响应将为记录类型 CNAME。 仅在终结点的类型为“外部”时，才允许将终结点添加为 IPv4 或 IPv6 地址。
三种终结点寻址类型支持所有路由方法和监视设置。

### <a name="what-types-of-ip-addresses-can-i-use-when-adding-an-endpoint"></a>添加终结点时可以使用哪些类型的 IP 地址？

流量管理器可使用 IPv4 或 IPv6 地址指定终结点。 下面列出了一些限制：

- 不允许使用与保留的私有网络 IP 地址空间对应的地址。 其中包括 RFC 1918、RFC 6890、RFC 5737、RFC 3068、RFC 2544 和 RFC 5771 中包括地址
- 地址不能包含任何端口号（可在配置文件配置设置中指定要使用的端口）
- 同一配置文件中的两个终结点不能具有相同的目标 IP 地址

### <a name="can-i-use-different-endpoint-addressing-types-within-a-single-profile"></a>可以在单个配置文件中使用不同的终结点寻址类型吗？

不能，流量管理器不允许在单个配置文件中混合使用不同的终结点寻址类型（多值路由类型的配置文件除外，可在其中混合使用 IPv4 和 IPv6 寻址类型）

### <a name="what-happens-when-an-incoming-querys-record-type-is-different-from-the-record-type-associated-with-the-addressing-type-of-the-endpoints"></a>当传入查询的记录类型与与终结点寻址类型关联的记录类型不同时，会出现什么情况？

当收到针对配置文件的查询时，流量管理器首先会根据指定的路由方法和终结点的运行状况查找需要返回的终结点。 然后，在根据下表返回响应之前，它会查看传入查询中请求的记录类型以及与终结点关联的记录类型。

对于使用多值路由以外的任何路由方法的配置文件：

|传入的查询请求|     终结点类型|     提供的响应|
|--|--|--|
|ANY |    A/AAAA/CNAME |    目标终结点| 
|A |    A/CNAME |    目标终结点|
|A |    AAAA |    无数据 |
|AAAA |    AAAA/CNAME |    目标终结点|
|AAAA |    A |    无数据 |
|CNAME |    CNAME |    目标终结点|
|CNAME     |A/AAAA |    无数据 |
|

对于将路由方法设置为多值路由的配置文件：

|传入的查询请求|     终结点类型 |    提供的响应|
|--|--|--|
|ANY |    混合 A 和 AAAA |    目标终结点|
|A |    混合 A 和 AAAA |    仅 A 类型的目标终结点|
|AAAA    |混合 A 和 AAAA|     仅 AAAA 类型的目标终结点|
|CNAME |    混合 A 和 AAAA |    无数据 |

### <a name="can-i-use-a-profile-with-ipv4--ipv6-addressed-endpoints-in-a-nested-profile"></a>可以在嵌套配置文件中使用终结点采用 IPv4/IPv6 地址的配置文件吗？

可以，有一种例外情况：多值类型的配置文件无法成为嵌套配置文件中的父级配置文件。

### <a name="i-stopped-an-web-application-endpoint-in-my-traffic-manager-profile-but-i-am-not-receiving-any-traffic-even-after-i-restarted-it-how-can-i-fix-this"></a>我在流量管理器配置文件中停止了 Web 应用程序终结点，但即使重启后，也未收到任何流量。 如何解决此问题？

当 Azure Web 应用程序终结点停止时，流量管理器会停止检查其运行状况，仅在检测到终结点已重启后，才重新开始运行状况检查。 若要防止这种延迟，请在流量管理器配置文件中禁用该终结点，然后在重启该终结点后将其重新启用。

### <a name="can-i-use-traffic-manager-even-if-my-application-does-not-have-support-for-http-or-https"></a>是否即使应用程序不支持 HTTP 或 HTTPS，我也可以使用流量管理器？

是的。 可将 TCP 指定为监视协议，而流量管理器可以发起 TCP 连接，并等待终结点的响应。 如果终结点在超时期限内回复了连接请求并提供了用于建立连接的响应，则会将终结点标记为正常。

### <a name="what-specific-responses-are-required-from-the-endpoint-when-using-tcp-monitoring"></a>使用 TCP 监视时，需要终结点发出的哪些特定响应？

使用 TCP 监视时，流量管理器将通过在指定的端口上向终结点发送 SYN 请求来启动三次 TCP 握手。 然后，它会在一段时间（在超时设置中指定）内等待终结点的 SYN-ACK 响应。

- 如果在监视设置中指定的超时期限内收到 SYN-ACK 响应，则认为该终结点正常。 FIN 或 FIN-ACK 是流量管理器定期终止套接字时的预期响应。
- 如果在指定的超时后收到 SYN-ACK 响应，则流量管理器将以 RST 响应以重置连接。

### <a name="how-fast-does-traffic-manager-move-my-users-away-from-an-unhealthy-endpoint"></a>流量管理器将用户从不正常终结点中移出的速度有多快？

流量管理器提供多个设置，可帮助控制流量管理器配置文件的故障转移行为，如下所述：

- 可以通过将“探测间隔”设置为 10 秒，指定流量管理器要更频繁地探测终结点。 这可确保尽快检测到任何即将进入不正常状态的终结点。 
- 可以指定在经过多长时间后，运行状况检查请求超时（最小超时值为 5 秒）。
- 可以指定在发生多少次失败之后，将终结点标记为不正常。 此值最小为 0，在这种情况下，一旦终结点通不过第一次运行状况检查，就会将它标记为不正常。 但是，对容许失败次数使用最小值 0 可能会因为探测时发生的任何暂时性问题，而从轮换项目中取出终结点。
- 可将 DNS 响应的生存时间 (TTL) 最小指定为 0。 这样做意味着 DNS 解析程序无法缓存响应，并且每个新查询将获取包含流量管理器中最新运行状况信息的响应。

如果使用这些设置，流量管理器可在终结点进入不正常状态后的 10 秒内提供故障转移，并针对相应的配置文件发出 DNS 查询。

### <a name="how-can-i-specify-different-monitoring-settings-for-different-endpoints-in-a-profile"></a>如何在配置文件中为不同的终结点指定不同的监视设置？

流量管理器监视设置在配置文件级别指定。 如果只需对一个终结点使用不同的监视设置，可以使用[嵌套配置文件](traffic-manager-nested-profiles.md)的形式配置该终结点，这样，其监视设置就会不同于父配置文件。

### <a name="how-can-i-assign-http-headers-to-the-traffic-manager-health-checks-to-my-endpoints"></a>如何将 HTTP 标头分配给终结点的流量管理器运行状况检查？

可以在流量管理器对终结点启动的 HTTP(S) 运行状况检查中指定自定义标头。 若要指定自定义标头，可在配置文件级别（适用于所有终结点）指定，也可在终结点级别指定。 如果在两个级别定义了标头，则在终结点级别指定的标头将覆盖在配置文件级别指定的标头。
一个常见的用例是指定主机头，以便流量管理器请求可以正确地路由到多租户环境中托管的终结点。 另一个用例是从终结点的 HTTP(S) 请求日志中识别流量管理器请求

### <a name="what-host-header-do-endpoint-health-checks-use"></a>终结点运行状况检查使用什么主机头？

如果未提供自定义主机头设置，流量管理器使用的主机头则为配置文件中配置的终结点目标的 DNS 名称（如果可用）。

<!-- Not Available ### What are the IP addresses from which the health checks originate? on 17th Mar 2017-->
### <a name="how-many-health-checks-to-my-endpoint-can-i-expect-from-traffic-manager"></a>流量管理器预期会对终结点执行多少次运行状况检查？

对终结点执行的流量管理器运行状况检查次数取决于以下因素：

- 为监视间隔设置的值（间隔越小，则在任意给定时间进入终结点的请求就越多）。
- 运行状况检查的来源位置数（前面的“常见问题解答”中列出了这些检查的预期来源 IP 地址）。

### <a name="how-can-i-get-notified-if-one-of-my-endpoints-goes-down"></a>如果我的终结点发生故障，我如何得到通知？

流量管理器提供的指标之一是配置文件中的终结点的运行状况状态。 可以将此指标作为配置文件中所有终结点的聚合进行查看（例如，75% 的终结点正常运行），也可以在每终结点级别查看此指标。 流量管理器指标是通过 Azure Monitor 公开的，并且，当终结点的运行状况状态发生更改时，你可以使用其[警报功能](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)获得通知。 有关更多详细信息，请参阅[流量管理器指标和警报](traffic-manager-metrics-alerts.md)。  

## <a name="traffic-manager-nested-profiles"></a>流量管理器嵌套式配置文件

### <a name="how-do-i-configure-nested-profiles"></a>如何配置嵌套式配置文件？

可以使用 Azure Resource Manager、经典 Azure REST API、Azure PowerShell cmdlet 和跨平台 Azure CLI 命令配置嵌套式流量管理器配置文件。 也支持通过新 Azure 门户配置这些配置文件。

### <a name="how-many-layers-of-nesting-does-traffic-manger-support"></a>流量管理器支持多少层嵌套？

配置文件的嵌套最深可达 10 层。 不允许使用“循环”。

### <a name="can-i-mix-other-endpoint-types-with-nested-child-profiles-in-the-same-traffic-manager-profile"></a>能否在同一流量管理器配置文件中将其他终结点类型与嵌套式子配置文件混合在一起使用？

是的。 至于如何在一个配置文件中组合使用不同类型的终结点，并无任何限制。

### <a name="how-does-the-billing-model-apply-for-nested-profiles"></a>嵌套式配置文件如何应用计费模型？

使用嵌套式配置文件在价格方面不会给你带来负面影响。

流量管理器计费分两部分：终结点运行状况检查以及数百万的 DNS 查询

* 终结点运行状况检查：在父配置文件中将子配置文件配置为终结点时，不对子配置文件收费。 在子配置文件中监视终结点按正常方式计费。
* DNS 查询：每个查询仅统计一次。 对父配置文件执行查询时，如果返回了子配置文件中的终结点，则仅统计父配置文件。

如需完整的详细信息，请参阅[流量管理器定价页](https://www.azure.cn/pricing/details/traffic-manager/)。

### <a name="is-there-a-performance-impact-for-nested-profiles"></a>嵌套式配置文件是否会造成性能影响？

否。 使用嵌套式配置文件不会造成性能影响。

在处理每个 DNS 查询时，流量管理器名称服务器会在内部遍历配置文件层次结构。 对父配置文件执行 DNS 查询可能会收到终结点来自子配置文件的 DNS 响应。 不管使用的是单个配置文件还是嵌套式配置文件，都只使用一条 CNAME 记录。 不需要在层次结构中为每个配置文件创建一条 CNAME 记录。

### <a name="how-does-traffic-manager-compute-the-health-of-a-nested-endpoint-in-a-parent-profile"></a>在父配置文件中，流量管理器如何计算嵌套式终结点的运行状况？

父配置文件不会针对子配置文件直接执行运行状况检查。 子配置文件的终结点运行状况用于计算子配置文件的总体运行状况。 此信息在嵌套式配置文件层次结构中向上传播，确定嵌套式终结点的运行状况。 父配置文件使用此聚合运行状况信息确定是否可将流量定向到子配置文件。

下表描述了针对嵌套式终结点执行的流量管理器运行状况检查的行为。

| 子配置文件监视器状态 | 父级终结点监视器状态 | 注释 |
| --- | --- | --- |
| 已禁用。 子配置文件已禁用。 |已停止 |父级终结点状态为“已停止”，而不是“已禁用”。 “已禁用”状态保留用于指示已在父配置文件中禁用了终结点。 |
| 已降级。 至少一个子配置文件终结点处于“已降级”状态。 |联机：子配置文件中处于“联机”状态的终结点的数目至少是 MinChildEndpoints 的值。<br />正在检查终结点：子配置文件中处于“联机”和“正在检查终结点”状态的终结点的数目至少是 MinChildEndpoints 的值。<br />已降级：其他。 |流量路由到状态为“正在检查终结点”的终结点。 如果将 MinChildEndpoints 设置得过高，终结点将始终处于降级状态。 |
| 联机。 至少一个子配置文件终结点处于“联机”状态。 没有任何终结点处于“已降级”状态。 |见上。 | |
| 正在检查终结点。 至少一个子配置文件终结点处于“正在检查终结点”状态。 没有任何终结点处于“联机”或“已降级”状态 |同上。 | |
| 非活动。 所有子配置文件终结点都处于“Disabled”或“Stopped”状态，除非这是没有终结点的配置文件。 |已停止 | |

## <a name="next-steps"></a>后续步骤：

- 详细了解流量管理器[终结点监视和自动故障转移](../traffic-manager/traffic-manager-monitoring.md)。
- 详细了解流量管理器[流量路由方法](../traffic-manager/traffic-manager-routing-methods.md)。

<!-- Update_Description: update meta properties, wording update, update link -->