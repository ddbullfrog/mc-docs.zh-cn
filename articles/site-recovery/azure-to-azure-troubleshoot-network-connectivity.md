---
title: 使用 Azure Site Recovery 对 Azure 到 Azure 灾难恢复的连接进行故障排除
description: 排查 Azure VM 灾难恢复中的连接问题
manager: rochakm
ms.topic: how-to
origin.date: 04/06/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 9b269faead8c7123cb8758a83cb82ad5f1ffb3d6
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127995"
---
# <a name="troubleshoot-azure-to-azure-vm-network-connectivity-issues"></a>排查 Azure 到 Azure VM 网络连接性问题

本文介绍了在不同区域之间复制和恢复 Azure 虚拟机 (VM) 时与网络连接相关的常见问题。 有关网络要求的详细信息，请参阅[复制 Azure VM 的连接性要求](azure-to-azure-about-networking.md)。

要使 Site Recovery 复制正常运行，必须具有从 VM 到特定 URL 或 IP 范围的出站连接。 如果 VM 位于防火墙后或使用网络安全组 (NSG) 规则来控制出站连接，则可能会遇到以下问题之一。

<!--MOONCAKE: REMOVE US GOVERMENT COLUMN DETAILS-->

| **Name** | **Azure 中国世纪互联** | **说明** |
| ------------------------- | -------------------------------------------- | ----------- |
| 存储                   | `*.blob.core.chinacloudapi.cn`                  | 必需，以便从 VM 将数据写入到源区域中的缓存存储帐户。 如果你知道 VM 的所有缓存存储帐户，则可以对特定存储帐户 URL 使用允许列表。 例如，使用 `cache1.blob.core.chinacloudapi.cn` 和 `cache2.blob.core.chinacloudapi.cn` 而不是 `*.blob.core.chinacloudapi.cn`。 |
| Azure Active Directory    | `login.chinacloudapi.cn`                | 对于 Site Recovery 服务 URL 的授权和身份验证而言是必需的。 |
| 复制               | `*.hypervrecoverymanager.windowsazure.cn` | 必需，以便从 VM 进行 Site Recovery 服务通信。 如果防火墙代理支持 IP，则可以使用相应的“Site Recovery IP”。 |
| 服务总线               | `*.servicebus.chinacloudapi.cn`                 | 必需，以便从 VM 写入 Site Recovery 监视和诊断数据。 如果防火墙代理支持 IP，则可以使用相应的“Site Recovery 监视 IP”。 |

<!--MOONCAKE: REMOVE US GOVERMENT COLUMN DETAILS-->

## <a name="outbound-connectivity-for-site-recovery-urls-or-ip-ranges-error-code-151037-or-151072"></a>Site Recovery URL 或 IP范围的出站连接（错误代码 151037 或 151072）

### <a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195"></a>问题 1：未能向 Site Recovery 注册 Azure 虚拟机 (151195)

#### <a name="possible-cause"></a>可能的原因

由于域名系统 (DNS) 解析失败，无法与 Site Recovery 终结点建立连接。 在重新保护期间，当你对 VM 进行故障转移但无法从灾难恢复 (DR) 区域访问 DNS 服务器时，此问题更为常见。

#### <a name="resolution"></a>解决方法

如果使用的是自定义 DNS，请确保可以从灾难恢复区域访问 DNS 服务器。

若要检查 VM 是否使用自定义 DNS 设置，请执行以下操作：

1. 打开“虚拟机”并选择 VM。
1. 导航到 VM 的“设置”并选择“网络”。
1. 在“虚拟网络/子网”中，选择相应链接以打开虚拟网络的资源页。
1. 转到“设置”，然后选择“DNS 服务器” 。

尝试从虚拟机访问 DNS 服务器。 如果 DNS 服务器无法访问，请通过对 DNS 服务器进行故障转移或创建 DR 网络与 DNS 之间站点的行来使其可访问。

:::image type="content" source="./media/azure-to-azure-troubleshoot-errors/custom_dns.png" alt-text="com-error":::

### <a name="issue-2-site-recovery-configuration-failed-151196"></a>问题 2：Site Recovery 配置失败 (151196)

> [!NOTE]
> 如果 VM 位于“标准”内部负载均衡器之后，则默认情况下，它将无法访问 Office 365 IP（如 `login.chinacloudapi.cn`）。 可以将其更改为“基本”内部负载均衡器类型，也可以按照[使用 Azure CLI 在标准负载均衡器中配置负载均衡和出站规则](../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard#create-outbound-rule-configuration)一文中所述创建出站访问权限。

#### <a name="possible-cause"></a>可能的原因

无法建立到 Office 365 身份验证和标识 IP4 终结点的连接。

#### <a name="resolution"></a>解决方法

- Azure Site Recovery 需要访问 Office 365 IP 范围才能进行身份验证。
- 如果使用 Azure 网络安全组 (NSG) 规则/防火墙代理控制 VM 上的出站网络连接，请确保允许到 Office 365 IP 范围的通信。 创建一个基于 [Azure Active Directory (Azure AD) 服务标记](../virtual-network/security-overview.md#service-tags)的 NSG 规则，该规则允许访问与 Azure AD 对应的所有 IP 地址。
- 如果将来向 Azure AD 添加新地址，则需创建新的 NSG 规则。

### <a name="example-nsg-configuration"></a>NSG 配置示例

此示例演示如何为要复制的 VM 配置 NSG 规则。

- 如果使用 NSG 规则控制出站连接，请对所有必需的 IP 地址范围使用端口 443 的“允许 HTTPS 出站”规则。
- 此示例假设 VM 源位置是“中国东部”，目标位置是“中国北部”。

#### <a name="nsg-rules---china-east"></a>NSG 规则 - 中国东部

1. 为 NSG 创建 HTTPS 出站安全规则，如以下屏幕截图所示。 此示例使用“目标服务标记”：“Storage.ChinaEast”和“目标端口范围”：“443”。

    :::image type="content" source="./media/azure-to-azure-about-networking/storage-tag.png" alt-text="com-error":::

1. 为 NSG 创建 HTTPS 出站安全规则，如以下屏幕截图所示。 此示例使用“目标服务标记”：“AzureActiveDirectory”和“目标端口范围”：“443”。

    :::image type="content" source="./media/azure-to-azure-about-networking/aad-tag.png" alt-text="com-error":::

1. 与上述安全规则类似，为 NSG 上的“EventHub.chinanorth”（对应于目标位置）创建出站 HTTPS (443) 安全规则。 这样就可以访问 Site Recovery 监视功能。
1. 在 NSG 上为“AzureSiteRecovery”创建出站 HTTPS (443) 安全规则。 这样就可以在任何区域访问 Site Recovery 服务。
    
    <!--MOONCAKE: CUSTOMIZE, UPDATE CAREFULLY-->
    <!--MOONCAKE: CORRECT ON China North | 40.125.202.254 | 42.159.4.151 -->
    
    > [!NOTE]
    > 如果 Azure 中国的特定区域不支持 `AzureSiteRecovery` 服务标记，我们可以为对应于目标位置的 Site Recovery IP 创建出站 HTTPS (443) 安全规则：
    >
    > 例如： 
    >
    >  |**位置** | **Site Recovery IP 地址** |  **Site Recovery 监视 IP 地址**|
    >  |--- | --- | ---|
    >  |中国北部 | 40.125.202.254 | 42.159.4.151|
    >  
    > ![site-recovery-ip-address](./media/azure-to-azure-about-networking/site-recovery-ip-address-chenye.png)
    
    <!--MOONCAKE: CORRECT ON China North | 40.125.202.254 | 42.159.4.151-->
    <!--MOONCAKE: CUSTOMIZE, UPDATE CAREFULLY-->

#### <a name="nsg-rules---china-north"></a>NSG 规则 - 中国北部

对于此示例，这些 NSG 规则是必需的，用于在故障转移后启用从目标区域到源区域的复制：

<!--MOONCAKE: CUSTOMIZE ON "Storage"， Not Available on .ChinaEast -->

1. 为 Storage 创建 HTTPS 出站安全规则：

    - **目标服务标记** ： _存储_
    - **目标端口范围** ： _443_

1. 为 AzureActiveDirectory 创建 HTTPS 出站安全规则。

    - **目标服务标记** ： _AzureActiveDirectory_
    - **目标端口范围** ： _443_

1. 与上述安全规则类似，为 NSG 上的“EventHub.ChinaEast”（对应于源位置）创建出站 HTTPS (443) 安全规则。 这样就可以访问 Site Recovery 监视功能。
1. 在 NSG 上为“AzureSiteRecovery”创建出站 HTTPS (443) 安全规则。 这样就可以在任何区域访问 Site Recovery 服务。
    
    <!--MOONCAKE: CUSTOMIZE, UPDATE CAREFULLY-->
    <!--MOONCAKE: CORRECT ON China East | 42.159.205.45 | 42.159.132.40 -->
    
    > [!NOTE]
    > 如果 Azure 中国的特定区域不支持 `AzureSiteRecovery` 服务标记，则可以为对应于源位置的 Site Recovery IP 创建出站 HTTPS (443) 安全规则：
    >
    > 例如：
    > 
    >  |**位置** | **Site Recovery IP 地址** |  **Site Recovery 监视 IP 地址**|
    >  |--- | --- | ---|
    >  |中国东部 | 42.159.205.45 | 42.159.132.40|
    >
    
    <!--MOONCAKE: CORRECT ON China East | 42.159.205.45 | 42.159.132.40 -->
    <!--MOONCAKE: CUSTOMIZE, UPDATE CAREFULLY-->


### <a name="issue-3-site-recovery-configuration-failed-151197"></a>问题 3：Site Recovery 配置失败 (151197)

#### <a name="possible-cause"></a>可能的原因

无法建立到 Azure Site Recovery 服务终结点的连接。

#### <a name="resolution"></a>解决方法

如果使用 Azure 网络安全组 (NSG) 规则/防火墙代理来控制计算机上的出站网络连接，则需要允许多个服务标记。 [了解详细信息](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags)。

### <a name="issue-4-azure-to-azure-replication-failed-when-the-network-traffic-goes-through-on-premises-proxy-server-151072"></a>问题 4：当网络流量通过本地代理服务器时，Azure 到 Azure 的复制失败 (151072)

#### <a name="possible-cause"></a>可能的原因

自定义代理设置无效，并且 Azure Site Recovery 移动服务代理未在 Internet Explorer (IE) 中自动检测到代理设置。

#### <a name="resolution"></a>解决方法

1. 移动服务代理通过 Windows 上的 IE 和 Linux 上的 `/etc/environment` 检测代理设置。
1. 如果只想对 Azure Site Recovery 移动服务设置代理，可在位于以下路径的 ProxyInfo.conf 中提供代理详细信息：

    - **Linux** ：`/usr/local/InMage/config/`
    - **Windows** ：`C:\ProgramData\Microsoft Azure Site Recovery\Config`

1. ProxyInfo.conf 应包含采用以下 INI 格式的代理设置：

    ```plaintext
    [proxy]
    Address=http://1.2.3.4
    Port=567
    ```

> [!NOTE]
> Azure Site Recovery 移动服务代理仅支持未经身份验证的代理。

### <a name="fix-the-problem"></a>解决问题

若要允许[所需 URL](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) 或[所需 IP 范围](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags)，请按照[网络指南文档](./azure-to-azure-about-networking.md)中的步骤进行操作。

## <a name="next-steps"></a>后续步骤

[将 Azure VM 复制到另一个 Azure 区域](azure-to-azure-how-to-enable-replication.md)

<!-- Update_Description: update meta properties, wording update, update link -->