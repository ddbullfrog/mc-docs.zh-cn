---
title: 排查 Azure VM 连接性问题
manager: digimobile
audience: ITPro
ms.topic: troubleshooting
ms.service: virtual-network
localization_priority: Normal
origin.date: 08/29/2019
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: no
ms.testdate: 10/05/2020
ms.author: v-yeche
ms.openlocfilehash: cd834224f89b7a81b03569edccaedfefbfad5694
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564548"
---
# <a name="troubleshoot-azure-vm-connectivity-problems"></a>排查 Azure VM 连接性问题

本文有助于管理员诊断和解决影响 Azure 虚拟机 (VM) 的连接问题。

## <a name="problems"></a>问题

- [通过资源管理器部署的 Azure VM 不能连接到同一虚拟网络中的另一个 Azure VM](#azure-vm-cannot-connect-to-another-azure-vm-in-same-virtual-network)。
- [Azure VM 不能连接到同一虚拟网络中 Azure VM 的第二个网络适配器](#azure-vm-cannot-connect-to-the-second-network-adapter-of-an-azure-vm-in-same-virtual-network)。
    
    <!--Not Available on - [An Azure VM can't connect to the internet](#azure-vm-cannot-connect-to-the-internet)-->

若要解决这些问题，请按以下部分的步骤操作。

## <a name="resolution"></a>解决方法

### <a name="azure-vm-cannot-connect-to-another-azure-vm-in-same-virtual-network"></a>Azure VM 无法连接到同一虚拟网络中的另一个 Azure VM

#### <a name="step-1-verify-that-vms-can-communicate-with-each-other"></a>步骤 1：验证 VM 可以互相通信。

1. 将 TCping 下载到源 VM。
2. 打开“命令提示”窗口。
3. 导航到 TCping 所下载到的文件夹。
4. 使用以下命令从源 VM ping 目标：

    :::image type="content" source="media/troubleshoot-vm-connectivity/tcping.png" alt-text="屏幕截图显示一个连续对 IP 地址执行 ping 操作的命令提示符窗口。":::

    ```cmd
    tcping64.exe -t <destination VM address> 3389
    ```

> [!TIP]
> 如果 ping 测试成功，请转到步骤 3。 否则，请转到下一步。

#### <a name="step-2-check-the-network-security-group-settings"></a>步骤 2：检查“网络安全组”设置。

针对每个 VM，检查默认的“入站端口规则”（“允许 VNet 入站”和“允许负载均衡器入站”）。 确保还检查在优先级较低的规则下没有列出匹配的阻止规则。

> [!NOTE]
> 编号较小的规则会先进行匹配。 例如，如果有优先级为 1000 和 6500 的规则，则优先级为 1000 的规则会先进行匹配。

然后，尝试再次从源 VM ping 目标：

```cmd
tcping64.exe -t <destination VM address> 3389
```

#### <a name="step-3-check-whether-you-can-connect-to-the-destination-vm-by-using-remote-desktop-or-ssh"></a>步骤 3：检查是否可以通过远程桌面或 SSH 连接到目标 VM。

若要通过远程桌面进行连接，请执行以下步骤。

**Windows**：

1. 登录到 Azure 门户。
2. 在左侧菜单中，选择“虚拟机”  。
3. 在列表中选择虚拟机。
4. 在虚拟机的页面上，选择“连接”  。

有关详细信息，请参阅[如何连接并登录到运行 Windows 的 Azure 虚拟机](/virtual-machines/windows/connect-logon)。

Linux  ：

有关详细信息，请参阅[连接到 Azure 中的 Linux VM](/virtual-machines/linux/quick-create-portal)。

如果远程桌面或 SSH 连接成功，请转到下一步。

#### <a name="step-4-perform-a-connectivity-check"></a>步骤 4：执行连接性检查。

在源 VM 上运行连接性检查，检查响应。

**Windows**：[使用 PowerShell 通过 Azure 网络观察程序检查连接性](/network-watcher/network-watcher-connectivity-powershell)

Linux  ：[使用 Azure CLI 2.0 通过 Azure 网络观察程序检查连接性](/network-watcher/network-watcher-connectivity-cli)

以下是示例响应：

```
ConnectionStatus : Unreachable
AvgLatencyInMs   :
MinLatencyInMs   :
MaxLatencyInMs   :
ProbesSent       : 100
ProbesFailed     : 100
Hops             : [
                     {
                       "Type": "Source",
                       "Id": "c5222ea0-3213-4f85-a642-cee63217c2f3",
                       "Address": "10.1.1.4",
                       "ResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGrou
                   ps/ContosoRG/providers/Microsoft.Network/networkInterfaces/appNic0/ipConfigurat
                   ions/ipconfig1",
                       "NextHopIds": [
                         "9283a9f0-cc5e-4239-8f5e-ae0f3c19fbaa"
                       ],
                       "Issues": []
                     },
                     {
                       "Type": "VirtualAppliance",
                       "Id": "9283a9f0-cc5e-4239-8f5e-ae0f3c19fbaa",
                       "Address": "10.1.2.4",
                       "ResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGrou
                   ps/ContosoRG/providers/Microsoft.Network/networkInterfaces/fwNic/ipConfiguratio
                   ns/ipconfig1",
                       "NextHopIds": [
                         "0f1500cd-c512-4d43-b431-7267e4e67017"
                       ],
                       "Issues": []
                     },
```

#### <a name="step-5-fix-the-issue-in-the-connectivity-check-result"></a>步骤 5：修复连接性检查结果中的问题。

1. 在收到的连接性检查响应的“跃点”部分，检查列出的**问题**。 

    :::image type="content" source="media/troubleshoot-vm-connectivity/connectivity-response.png" alt-text="屏幕截图显示一个连续对 IP 地址执行 ping 操作的命令提示符窗口。":::

2. 在下表中找到相应的解决方法，按指示的步骤解决问题。

    |问题类型  |Value  |解决操作 |
    |---------|---------|---------|
    |NetworkSecurityRule|阻止 NSG 的名称|可以[删除 NSG 规则](/virtual-network/manage-network-security-group#delete-a-security-rule)，也可以按[此处](/virtual-network/manage-network-security-group#change-a-security-rule)的说明修改该规则。|
    |UserDefinedRoute     |   阻止 UDR 的名称      | 如果不需要此路由，请删除 UDR。 如果不能删除此路由，请使用相应的地址前缀和下一跃点更新此路由。 也可调整“网络虚拟设备”，以适当方式转发流量。 有关详细信息，请参阅：[虚拟网络流量路由](/virtual-network/virtual-networks-udr-overview)和[使用 PowerShell 通过路由表路由网络流量](/virtual-network/virtual-network-create-udr-arm-ps)。|
    |CPU    |    使用情况     |     按[运行 Linux 或 Windows 的 Azure 虚拟机常规性能故障排除](https://support.microsoft.com/en-in/help/3150851/generic-performance-troubleshooting-for-azure-virtual-machine-running)中介绍的这些建议操作。|
    |内存    |      使用情况   |    按[运行 Linux 或 Windows 的 Azure 虚拟机常规性能故障排除](https://support.microsoft.com/en-in/help/3150851/generic-performance-troubleshooting-for-azure-virtual-machine-running)中介绍的建议操作。|
    |来宾防火墙    |      防火墙阻止的名称   |      执行以下步骤：[打开或关闭 Windows Defender 防火墙](https://support.microsoft.com/help/4028544/windows-turn-windows-firewall-on-or-off)。|
    |DNS 解析     |    DNS 的名称     |    执行以下步骤：[Azure DNS 故障排除指南](/dns/dns-troubleshoot)和 [Azure 虚拟网络中资源的名称解析](/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances)。     |
    |套接字错误    |      不适用   |     指定的端口已由另一应用程序使用。 请尝试使用另一端口。    |

3. 再次运行连接性检查，确定问题是否已解决。

### <a name="azure-vm-cannot-connect-to-the-second-network-adapter-of-an-azure-vm-in-same-virtual-network"></a>Azure VM 不能连接到同一虚拟网络中 Azure VM 的第二个网络适配器

#### <a name="step-1-make-sure-that-the-second-network-adapter-is-enabled-to-talk-outside-the-subnet"></a>步骤 1：确保启用第二个网络适配器，以便在子网外部通信。

默认情况下，辅助网络适配器（也称为网络接口卡或网络适配器）未配置为拥有默认网关。 因此，辅助适配器上的通信流会限制在同一子网内。

:::image type="content" source="media/troubleshoot-vm-connectivity/ipconfig.png" alt-text="屏幕截图显示一个连续对 IP 地址执行 ping 操作的命令提示符窗口。":::

如果用户要启用辅助网络适配器，以在自己的子网外部通信，则必须在路由表中添加一个条目来配置网关。 为此，请执行以下步骤：

1. 在配置了第二个网络适配器的 VM 上，以管理员身份打开一个命令提示符窗口。
2. 运行以下命令，在路由表中添加此条目：

    ```cmd
    Route add 0.0.0.0 mask 0.0.0.0 -p <Gateway IP>
    ```

    例如，如果第二个 IP 地址为 192.168.0.4，则网关 IP 应该为 192.168.0.1。 必须运行以下命令：

    ```cmd
    Route add 0.0.0.0 mask 0.0.0.0 -p 192.168.0.1
    ```

3. 运行 route print。 如果条目已成功添加，则会看到一个如下所示的条目：

    :::image type="content" source="media/troubleshoot-vm-connectivity/iproute.png" alt-text="屏幕截图显示一个连续对 IP 地址执行 ping 操作的命令提示符窗口。":::

现在，尝试连接到辅助网络适配器。 如果连接仍未成功，请转到下一步。

#### <a name="step-2-check-nsg-settings-for-the-network-adapters"></a>步骤 2：检查网络适配器的 NSG 设置。

针对主网络适配器和辅助网络适配器，检查两个网络适配器上的默认“入站端口规则”（“允许 VNet 入站”、“允许负载均衡器入站”）。 还应确保优先级较低的规则下没有匹配的阻止规则。

:::image type="content" source="media/troubleshoot-vm-connectivity/nsg.png" alt-text="屏幕截图显示一个连续对 IP 地址执行 ping 操作的命令提示符窗口。":::

#### <a name="step-3-run-a-connectivity-check-to-the-secondary-network-adapter"></a>步骤 3：运行针对辅助网络适配器的连接性检查。

1. 运行针对辅助网络适配器的连接性检查。
2. 对整个环境运行连接性检查，确保进程能够端到端运行。

若要详细了解如何运行连接性检查，请参阅以下文章：

**Windows**：[使用 PowerShell 通过 Azure 网络观察程序检查连接性](/network-watcher/network-watcher-connectivity-powershell)

Linux  ：[使用 Azure CLI 2.0 通过 Azure 网络观察程序检查连接性](/network-watcher/network-watcher-connectivity-cli)。

以下是示例响应：

```
ConnectionStatus : Unreachable
AvgLatencyInMs   : 
MinLatencyInMs   : 
MaxLatencyInMs   : 
ProbesSent       : 100
ProbesFailed     : 100
Hops             : [
                     {
                       "Type": "Source",
                       "Id": "c5222ea0-3213-4f85-a642-cee63217c2f3",
                       "Address": "10.1.1.4",
                       "ResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGrou
                   ps/ContosoRG/providers/Microsoft.Network/networkInterfaces/appNic0/ipConfigurat
                   ions/ipconfig1",
                       "NextHopIds": [
                         "9283a9f0-cc5e-4239-8f5e-ae0f3c19fbaa"
                       ],
                       "Issues": []
                     },
                     {
                       "Type": "VirtualAppliance",
                       "Id": "9283a9f0-cc5e-4239-8f5e-ae0f3c19fbaa",
                       "Address": "10.1.2.4",
                       "ResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGrou
                   ps/ContosoRG/providers/Microsoft.Network/networkInterfaces/fwNic/ipConfiguratio
                   ns/ipconfig1",
                       "NextHopIds": [
                         "0f1500cd-c512-4d43-b431-7267e4e67017"
                       ],
                       "Issues": []
                     },
```

#### <a name="step-4-refer-the-table-under-step-5-and-follow-these-steps-to-resolve-the-issues"></a>步骤 4：参阅[步骤 5](#step-5-fix-the-issue-in-the-connectivity-check-result) 下的表，按这些步骤解决问题。

<!--Not Available on ### Azure VM cannot connect to the internet-->
<!--Not Available on portal
7. Select the **Read/Write** option at the top of the portal.
8. Select the **Edit** option.
-->
<!--Not Available on [Resource Explorer portal](https://resources.azure.com/)-->

## <a name="next-steps"></a>后续步骤

[排查 Azure VM 间的连接问题](/virtual-network/virtual-network-troubleshoot-connectivity-problem-between-vms)

<!-- Update_Description: update meta properties, wording update, update link -->