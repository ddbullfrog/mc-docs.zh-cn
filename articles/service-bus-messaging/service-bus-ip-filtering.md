---
title: 配置 Azure 服务总线的 IP 防火墙规则
description: 如何使用防火墙规则允许从特定 IP 地址连接到 Azure 服务总线。
ms.topic: article
origin.date: 06/23/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: yes
ms.testdate: 08/17/2020
ms.author: v-yeche
ms.openlocfilehash: a1600b341b1fb8e3b0f97a9e5efa8dab6a3757fb
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127833"
---
# <a name="allow-access-to-azure-service-bus-namespace-from-specific-ip-addresses-or-ranges"></a>允许从特定 IP 地址或范围访问 Azure 服务总线命名空间
默认情况下，只要请求附带有效的身份验证和授权，就可以从 Internet 访问服务总线命名空间。 使用 IP 防火墙，可以将其进一步限制为采用 CIDR（无类域间路由）表示法的一组 IPv4 地址或一个 IPv4 地址。

<!--Not Available on [CIDR (Classless Inter-Domain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)-->

在仅应从某些知名站点访问 Azure 服务总线的情况下，此功能很有用。 可以通过防火墙规则来配置规则，以便接受来自特定 IPv4 地址的流量。 例如，如果将服务总线与 [Azure Express Route][express-route] 配合使用，则可创建 **防火墙规则** ，以便仅允许来自本地基础结构 IP 地址或企业 NAT 网关地址的流量。 

> [!IMPORTANT]
> 防火墙和虚拟网络仅在服务总线的 **高级** 层中受支持。 如果无法升级到 **高级** 层，我们建议保护共享访问签名 (SAS) 令牌的安全，只与已获授权的用户共享。 有关 SAS 身份验证的信息，请参阅[身份验证和授权](service-bus-authentication-and-authorization.md#shared-access-signature)。

## <a name="ip-firewall-rules"></a>IP 防火墙规则
IP 防火墙规则在服务总线命名空间级别应用。 因此，这些规则适用于通过任何受支持协议从客户端发出的所有连接。 如果某 IP 地址与服务总线命名空间上的允许 IP 规则不匹配，则将拒绝来自该地址的任何连接尝试并将其标记为“未经授权”。 响应不会提及 IP 规则。 IP 筛选器规则将按顺序应用，与 IP 地址匹配的第一个规则决定了将执行接受操作还是执行拒绝操作。

>[!WARNING]
> 实施防火墙规则可以组织其他 Azure 服务与服务总线进行交互。
>
> 实施 IP 筛选（防火墙规则）时，受信任的 Azure 服务不受支持，但很快就会变得可用。
>
> 不适用于 IP 筛选的常见 Azure 方案（请注意，该列表内容并不详尽）-
> - 与 Azure 事件网格的集成
> - Azure IoT 中心路由
> - Azure IoT Device Explorer
>
> 以下 Azure 服务必须在虚拟网络中
> - Azure 应用服务
> - Azure Functions
> - Azure Monitor（诊断设置）

## <a name="use-azure-portal"></a>使用 Azure 门户
本部分介绍了如何使用 Azure 门户为服务总线命名空间创建 IP 防火墙规则。 

1. 在 [Azure 门户](https://portal.azure.cn)中，导航到“服务总线命名空间”。
2. 在左侧菜单上，选择“设置”下的“网络”选项 。  

    > [!NOTE]
    > 只会为“高级”命名空间显示“网络”选项卡 。  

    默认情况下，“选定网络”选项处于选中状态。 如果未在此页上添加至少一个 IP 防火墙规则或虚拟网络，则可以通过公共 Internet（使用访问密钥）访问该命名空间。

    :::image type="content" source="./media/service-bus-ip-filtering/default-networking-page.png" alt-text="网络页面 - 默认" lightbox="./media/service-bus-ip-filtering/default-networking-page.png":::

    如果你选择“所有网络”选项，你的服务总线命名空间将接受来自 IP 地址的连接。 此默认设置等效于接受 0.0.0.0/0 IP 地址范围的规则。 

    :::image type="content" source="./media/service-bus-ip-filtering/firewall-all-networks-selected.png" alt-text="网络页面 - 默认":::
1. 若要仅允许从指定的 IP 地址进行访问，请选择“选定网络”选项（如果尚未选择）。 在“防火墙”部分中执行以下步骤：
    1. 选择“添加客户端 IP 地址”选项，使当前客户端 IP 可以访问命名空间。 
    2. 对于“地址范围”，请输入某个特定的 IPv4 地址或以 CIDR 表示法表示的 IPv4 地址范围。 
    3. 指定是否要“允许受信任的 Microsoft 服务绕过此防火墙”。 
    
        <!--CORRECT ON trusted Microsoft services on UI-->

        > [!WARNING]
        > 如果选择“选定的网络”选项但未指定 IP 地址或地址范围，则服务将允许来自所有网络的流量。 

        :::image type="content" source="./media/service-bus-ip-filtering/firewall-selected-networks-trusted-access-disabled.png" alt-text="网络页面 - 默认":::
3. 在工具栏上选择“保存”，保存这些设置。 请等待几分钟，直到门户通知中显示确认消息。

    > [!NOTE]
    > 若要限制对特定虚拟网络的访问，请参阅[允许从特定网络访问](service-bus-service-endpoints.md)。

## <a name="use-resource-manager-template"></a>使用 Resource Manager 模板
本部分提供了一个用于创建虚拟网络和防火墙规则的 Azure 资源管理器模板示例。

以下资源管理器模板支持向现有服务总线命名空间添加虚拟网络规则。

模板参数：

- ipMask 是单个 IPv4 地址或者是以 CIDR 表示法表示的一个 IP 地址块。 例如，在 CIDR 表示法中，70.37.104.0/24 表示从 70.37.104.0 到 70.37.104.255 的 256 个 IPv4 地址，其中 24 表示范围的有效前缀位数。

> [!NOTE]
> 虽然不可能具有拒绝规则，但 Azure 资源管理器模板的默认操作设置为“允许”，不限制连接。
> 制定虚拟网络或防火墙规则时，必须更改“defaultAction”
> 
> from
> ```json
> "defaultAction": "Allow"
> ```
> to
> ```json
> "defaultAction": "Deny"
> ```
>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "servicebusNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Service Bus namespace"
        }
      },
      "location": {
        "type": "string",
        "metadata": {
          "description": "Location for Namespace"
        }
      }
    },
    "variables": {
      "namespaceNetworkRuleSetName": "[concat(parameters('servicebusNamespaceName'), concat('/', 'default'))]",
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('servicebusNamespaceName')]",
        "type": "Microsoft.ServiceBus/namespaces",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Premium",
          "tier": "Premium"
        },
        "properties": { }
      },
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[variables('namespaceNetworkRuleSetName')]",
        "type": "Microsoft.ServiceBus/namespaces/networkruleset",
        "dependsOn": [
          "[concat('Microsoft.ServiceBus/namespaces/', parameters('servicebusNamespaceName'))]"
        ],
        "properties": {
          "virtualNetworkRules": [<YOUR EXISTING VIRTUAL NETWORK RULES>],
          "ipRules": 
          [
            {
                "ipMask":"10.1.1.1",
                "action":"Allow"
            },
            {
                "ipMask":"11.0.0.0/24",
                "action":"Allow"
            }
          ],
          "trustedServiceAccessEnabled": false,          
          "defaultAction": "Deny"
        }
      }
    ],
    "outputs": { }
  }
```

若要部署模板，请按照 [Azure 资源管理器][lnk-deploy]的说明进行操作。

## <a name="next-steps"></a>后续步骤

若要限制服务总线到 Azure 虚拟网络的访问，请参阅以下链接：

- [服务总线的虚拟网络服务终结点][lnk-vnet]

<!-- Links -->

[lnk-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[lnk-vnet]: service-bus-service-endpoints.md
[express-route]:  ../expressroute/expressroute-faqs.md#supported-services

<!-- Update_Description: update meta properties, wording update, update link -->