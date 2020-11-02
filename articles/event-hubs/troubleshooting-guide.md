---
title: 排查连接问题 - Azure 事件中心 | Microsoft Docs
description: 本文介绍如何排查 Azure 事件中心的连接问题。
ms.topic: article
origin.date: 06/23/2020
ms.date: 08/21/2020
ms.author: v-tawe
ms.openlocfilehash: 9496634e71476da8e313f68a9b358b8cc9d2ece1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103531"
---
# <a name="troubleshoot-connectivity-issues---azure-event-hubs"></a>排查连接问题 - Azure 事件中心
客户端应用程序无法连接到事件中心的原因有很多。 你遇到的连接问题可能是永久性的，也可能是暂时性的。 如果问题一直发生（永久性的），则可能需要检查连接字符串、组织的防火墙设置、IP 防火墙设置、网络安全设置（服务终结点、专用终结点等），等等。 对于暂时性问题，升级到最新版本的 SDK、运行命令来检查丢弃的数据包以及获取网络跟踪可能有助于解决问题。 

本文提供了排查 Azure 事件中心的连接问题的技巧。 

## <a name="troubleshoot-permanent-connectivity-issues"></a>排查永久性连接问题
如果应用程序根本无法连接到事件中心，请按此部分的步骤操作来解决问题。 

### <a name="check-if-there-is-a-service-outage"></a>检查是否存在服务中断
在 [Azure 服务状态站点](https://azure.microsoft.com/status/)上检查是否存在 Azure 事件中心服务中断的情况。

### <a name="verify-the-connection-string"></a>验证连接字符串 
验证你使用的连接字符串是否正确。 请参阅[获取连接字符串](event-hubs-get-connection-string.md)，以便使用 Azure 门户、CLI 或 PowerShell 获取连接字符串。 

对于 Kafka 客户端，请验证是否正确配置了 producer.config 或 consumer.config 文件。 有关详细信息，请参阅[在事件中心内使用 Kafka 发送和接收消息](event-hubs-quickstart-kafka-enabled-event-hubs.md#send-and-receive-messages-with-kafka-in-event-hubs)。

### <a name="check-if-the-ports-required-to-communicate-with-event-hubs-are-blocked-by-organizations-firewall"></a>检查组织的防火墙是否阻止了与事件中心通信所需的端口
确认组织的防火墙未阻止用于与 Azure 事件中心通信的端口。 请查看下表，了解与 Azure 事件中心通信需要打开哪些出站端口。 

| 协议 | 端口 | 详细信息 | 
| -------- | ----- | ------- | 
| AMQP | 5671 和 5672 | 请参阅 [AMQP 协议指南](../service-bus-messaging/service-bus-amqp-protocol-guide.md) | 
| HTTP、HTTPS | 80、443 |  |
| Kafka | 9093 | 请参阅[使用 Kafka 应用程序中的事件中心](event-hubs-for-kafka-ecosystem-overview.md)

下面是用于检查 5671 端口是否被阻止的示例命令。

```powershell
tnc <yournamespacename>.servicebus.chinacloudapi.cn -port 5671
```

在 Linux 上：

```shell
telnet <yournamespacename>.servicebus.chinacloudapi.cn 5671
```

### <a name="verify-that-ip-addresses-are-allowed-in-your-corporate-firewall"></a>验证你的企业防火墙中是否允许使用这些 IP 地址
使用 Azure 时，有时必须在公司防火墙或代理中允许特定的 IP 地址范围或 URL 才能访问你正在使用或尝试使用的所有 Azure 服务。 确认在事件中心使用的 IP 地址上是否允许该流量。 有关 Azure 事件中心使用的 IP 地址，请参阅 [Azure IP 范围和服务标记 - 中国云](https://www.microsoft.com/download/confirmation.aspx?id=57062)。

另外，请验证是否允许你的命名空间的 IP 地址。 若要查找允许你的连接的正确 IP 地址，请执行以下步骤：

1. 从命令提示符处运行以下命令： 

    ```
    nslookup <YourNamespaceName>.servicebus.chinacloudapi.cn
    ```
2. 记下 `Non-authoritative answer` 中返回的 IP 地址。 只有在你将命名空间还原到另一群集时，它才会更改。

如果对命名空间使用区域冗余，则需要执行一些额外步骤： 

1. 首先，在命名空间中运行 nslookup。

    ```
    nslookup <yournamespace>.servicebus.chinacloudapi.cn
    ```
2. 记下“非权威回答”部分中的名称，该名称采用下述格式之一： 

    ```
    <name>-s1.chinacloudapp.cn
    <name>-s2.chinacloudapp.cn
    <name>-s3.chinacloudapp.cn
    ```

3. 为带有后缀 s1、s2 和 s3 的每个实例运行 nslookup，以获取在三个可用性区域中运行的所有三个实例的 IP 地址。

<!-- ### Verify that AzureEventGrid service tag is allowed in your network security groups
If your application is running inside a subnet and there is an associated network security group, confirm whether the internet outbound is allowed or AzureEventGrid service tag is allowed. See [Virtual network service tags](../virtual-network/service-tags-overview.md) and search for `EventHub`.

### Check if the application needs to be running in a specific subnet of a vnet
Confirm that your application is running in a virtual network subnet that has access to the namespace. If it's not, run the application in the subnet that has access to the namespace or add the IP address of the machine on which application is running to the [IP firewall](event-hubs-ip-filtering.md). 

When you create a virtual network service endpoint for an event hub namespace, the namespace accepts traffic only from the subnet that's bound to the service endpoint. There is an exception to this behavior. You can add specific IP addresses in the IP firewall to enable access to the Event Hub public endpoint. For more information, see [Network service endpoints](event-hubs-service-endpoints.md).

### Check the IP Firewall settings for your namespace
Check that the public IP address of the machine on which the application is running isn't blocked by the IP firewall.  

By default, Event Hubs namespaces are accessible from internet as long as the request comes with valid authentication and authorization. With IP firewall, you can restrict it further to only a set of IPv4 addresses or IPv4 address ranges in CIDR (Classless Inter-Domain Routing) notation.

The IP firewall rules are applied at the Event Hubs namespace level. Therefore, the rules apply to all connections from clients using any supported protocol. Any connection attempt from an IP address that does not match an allowed IP rule on the Event Hubs namespace is rejected as unauthorized. The response does not mention the IP rule. IP filter rules are applied in order, and the first rule that matches the IP address determines the accept or reject action.

For more information, see [Configure IP firewall rules for an Azure Event Hubs namespace](event-hubs-ip-filtering.md). To check whether you have IP filtering, virtual network, or certificate chain issues, see [Troubleshoot network related issues](#troubleshoot-network-related-issues).

#### Find the IP addresses blocked by IP Firewall
Enable diagnostic logs for [Event Hubs virtual network connection events](event-hubs-diagnostic-logs.md#event-hubs-virtual-network-connection-event-schema) by following instructions in the [Enable diagnostic logs](event-hubs-diagnostic-logs.md#enable-diagnostic-logs). You will see the IP address for the connection that's denied.

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```
-->
<!-- ### Check if the namespace can be accessed using only a private endpoint
If the Event Hubs namespace is configured to be accessible only via private endpoint, confirm that the client application is accessing the namespace over the private endpoint. 

[Azure Private Link service](../private-link/private-link-overview.md) enables you to access Azure Event Hubs over a **private endpoint** in your virtual network. A private endpoint is a network interface that connects you privately and securely to a service powered by Azure Private Link. The private endpoint uses a private IP address from your virtual network, effectively bringing the service into your virtual network. All traffic to the service can be routed through the private endpoint, so no gateways, NAT devices, ExpressRoute or VPN connections, or public IP addresses are needed. Traffic between your virtual network and the service traverses over the Microsoft backbone network, eliminating exposure from the public Internet. You can connect to an instance of an Azure resource, giving you the highest level of granularity in access control.

For more information, see [Configure private endpoints](private-link-service.md). See the **Validate that the private endpoint connection works** section to confirm that a private endpoint is used. 
-->

### <a name="troubleshoot-network-related-issues"></a>排查网络相关问题
若要排查事件中心的网络相关问题，请执行以下步骤： 

浏览至 `https://<yournamespacename>.servicebus.chinacloudapi.cn/` 或使用 [wget](https://www.gnu.org/software/wget/)。 这可帮助检查是否存在 IP 筛选或虚拟网络或证书链问题（使用 Java SDK 时最常见）。

**成功消息** 的示例：

```xml
<feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
```

**失败错误消息** 的示例：

```json
<Error>
    <Code>400</Code>
    <Detail>
        Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
    </Detail>
</Error>
```

## <a name="troubleshoot-transient-connectivity-issues"></a>排查暂时性连接问题
如果遇到间歇性连接问题，请参阅以下部分来了解排查技巧。 

### <a name="use-the-latest-version-of-the-client-sdk"></a>使用最新版本的客户端 SDK
在较高的 SDK 版本（高于你所用版本）中，一些暂时性的连接问题可能已经修复。 确保在应用程序中使用最新版本的客户端 SDK。 SDK 通过新的/更新的功能和 bug 修复持续进行改进，因此请始终使用最新的包进行测试。 请查看发行说明，了解已修复的问题以及已添加/更新的功能。 

有关客户端 SDK 的信息，请参阅 [Azure 事件中心 - 客户端 SDK](sdks.md) 一文。 

### <a name="run-the-command-to-check-dropped-packets"></a>运行命令来检查丢弃的数据包
出现间歇性连接问题时，请运行以下命令，检查是否存在任何丢弃的数据包。 此命令会尝试通过服务每隔 1 秒建立 25 个不同的 TCP 连接。 然后，可以检查其中有多少成功/失败，还可以查看 TCP 连接延迟。 可以从[此处](https://docs.microsoft.com/sysinternals/downloads/psping)下载 `psping` 工具。

```shell
.\psping.exe -n 25 -i 1 -q <yournamespacename>.servicebus.chinacloudapi.cn:5671 -nobanner
```
如果使用的是其他工具（如 `tnc`、`ping` 等），则可以使用等效的命令。 

如果上述步骤没有帮助，请获取网络跟踪，并使用 [Wireshark](https://www.wireshark.org/) 之类的工具对其进行分析。 如果需要，请联系 [Microsoft 支持部门](https://support.azure.cn/)。

### <a name="service-upgradesrestarts"></a>服务升级/重启
由于后端服务升级和重启，可能会出现暂时性连接问题。 出现这种情况时，可能会看到以下症状： 

- 传入的消息/请求可能会减少。
- 日志文件可能包含错误消息。
- 应用程序可能会在几秒内断开与服务的连接。
- 可能会暂时限制请求。

如果应用程序代码使用 SDK，则重试策略已内置且处于活动状态。 应用程序会重新连接，此操作不会对应用程序/工作流产生重大影响。 捕获这些暂时性错误，后退然后重试调用，将确保代码能够从这些暂时性错误中复原。

## <a name="next-steps"></a>后续步骤
请参阅以下文章：

* [排查身份验证和授权问题](troubleshoot-authentication-authorization.md)
