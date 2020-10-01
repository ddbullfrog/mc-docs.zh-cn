---
title: Azure 防火墙规则处理逻辑
description: Azure 防火墙具有 NAT 规则、网络规则和应用程序规则。 规则是根据规则类型进行处理的。
services: firewall
ms.service: firewall
ms.topic: article
origin.date: 04/10/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.openlocfilehash: 898130163dad38495a217e385c1eb19a2ce4f552
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246794"
---
# <a name="configure-azure-firewall-rules"></a>配置 Azure 防火墙规则
在 Azure 防火墙上可以配置 NAT 规则、网络规则和应用程序规则。 处理规则集合时，会根据规则类型按优先级顺序（由低编号到高编号，从 100 到 65,000）进行。 规则集合名称只能包含字母、数字、下划线、句点或连字符。 该名称必须以字母或数字开头，并且以字母、数字或下划线结尾。 名称最大长度为 80 个字符。

最好在最初以 100 为增量（100、200、300，依此类推）设置规则集合优先级编号，这样在需要时就还有空间，可以添加更多的规则集合。

> [!NOTE]
> 如果启用基于威胁情报的筛选，则那些规则具有最高优先级，始终会首先处理。 在处理任何已配置的规则之前，威胁情报筛选可能会拒绝流量。 有关详细信息，请参阅 [Azure 防火墙基于威胁情报的筛选](threat-intel.md)。

## <a name="outbound-connectivity"></a>出站连接

### <a name="network-rules-and-applications-rules"></a>网络规则和应用程序规则

如果配置了网络规则和应用程序规则，则会在应用程序规则之前先按优先级顺序应用网络规则。 规则将终止。 因此，如果在网络规则中找到了匹配项，则不会处理其他规则。  如果没有网络规则匹配项，并且，如果协议是 HTTP、HTTPS 或 MSSQL，则应用程序规则会按优先级顺序评估数据包。 如果仍未找到匹配项，则会根据[基础结构规则集合](infrastructure-fqdns.md)评估数据包。 如果仍然没有匹配项，则默认情况下会拒绝该数据包。

## <a name="inbound-connectivity"></a>入站连接

### <a name="nat-rules"></a>NAT 规则

可以通过配置目标网络地址转换 (DNAT) 来启用入站 Internet 连接，如[教程：使用 Azure 门户通过 Azure Firewall DNAT 筛选入站流量](tutorial-firewall-dnat.md)中所述。 NAT 规则会在网络规则之前按优先级应用。 如果找到匹配项，则会添加一个隐式的对应网络规则来允许转换后的流量。 可以通过以下方法替代此行为：显式添加一个网络规则集合并在其中包含将匹配转换后流量的拒绝规则。

<!--Not Available on Application rules aren't applied for inbound connections. So if you want to filter inbound HTTP/S traffic, you should use Web Application Firewall (WAF). For more information, see [What is Azure Web Application Firewall?](../web-application-firewall/overview.md)-->

## <a name="examples"></a>示例

下面的示例显示了组合使用其中一些规则时的结果。

### <a name="example-1"></a>示例 1

<!--MOONCAKE CUSTOMIZATION: Change from google to qq--> 

由于存在匹配的网络规则，因此允许连接到 qq.com。

网络规则

- 操作：允许

|name  |协议  |源类型  |源  |目标类型  |目标地址  |目标端口|
|---------|---------|---------|---------|----------|----------|--------|
|Allow-web     |TCP|IP 地址|*|IP 地址|*|80,443

应用程序规则

- 操作：Deny

|name  |源类型  |源  |协议:端口|目标 FQDN|
|---------|---------|---------|---------|----------|----------|
|Deny-qq     |IP 地址|*|http:80,https:443|qq.com

**结果**

允许连接到 qq.com，因为该数据包符合 Allow-web 网络规则。 此时，规则处理停止。

<!--MOONCAKE CUSTOMIZATION: Change from google to qq--> 

### <a name="example-2"></a>示例 2

由于优先级较高的 Deny 网络规则集合阻止 SSH 流量，因此 SSH 流量被拒绝。

网络规则集合 1

- 姓名：Allow-collection
- 优先级：200
- 操作：允许

|name  |协议  |源类型  |源  |目标类型  |目标地址  |目标端口|
|---------|---------|---------|---------|----------|----------|--------|
|Allow-SSH     |TCP|IP 地址|*|IP 地址|*|22

网络规则集合 2

- 姓名：Deny-collection
- 优先级：100
- 操作：Deny

|name  |协议  |源类型  |源  |目标类型  |目标地址  |目标端口|
|---------|---------|---------|---------|----------|----------|--------|
|Deny-SSH     |TCP|IP 地址|*|IP 地址|*|22

**结果**

由于优先级较高的网络规则集合阻止 SSH 连接，因此 SSH 连接被拒绝。 此时，规则处理停止。

## <a name="rule-changes"></a>规则更改

如果更改规则以拒绝以前允许的流量，则会删除任何相关的现有会话。

## <a name="next-steps"></a>后续步骤

- 了解如何[部署和配置 Azure 防火墙](tutorial-firewall-deploy-portal.md)。

<!-- Update_Description: update meta properties, wording update, update link -->