---
title: 排查域和 TLS/SSL 证书问题
description: 查找在 Azure 应用服务中配置域或 TLS/SSL 证书时可能遇到的常见问题的解决方案。
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.topic: article
origin.date: 03/01/2019
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: seodec18
ms.openlocfilehash: 3b142f41b5a3493d44d4326be7c3e069667732c5
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170598"
---
# <a name="troubleshoot-domain-and-tlsssl-certificate-problems-in-azure-app-service"></a>排查 Azure 应用服务中的域和 TLS/SSL 证书问题

本文列出了为 Azure 应用服务中的 Web 应用配置域或 TLS/SSL 证书时可能遇到的常见问题。 此外，还描述了这些问题的可能原因和解决方案。

对于本文中的任何内容，如果需要更多帮助，可以联系 [MSDN 和 Stack Overflow 论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs)上的 Azure 专家。 或者，也可以提出 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/support/contact/)并选择“获取支持”。


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="certificate-problems"></a>证书问题

### <a name="you-cant-add-a-tlsssl-certificate-binding-to-an-app"></a>无法向应用添加 TLS/SSL 证书绑定 

#### <a name="symptom"></a>症状

添加 TLS 绑定时收到以下错误消息：

“未能添加 SSL 绑定。 无法设置现有 VIP 的证书，因为另一个 VIP 已使用该证书。”

#### <a name="cause"></a>原因

如果对跨多个应用的同一 IP 地址使用多个基于 IP 的 SSL 绑定，则可能会出现此问题。 例如，应用 A 具有采用旧证书的基于 IP 的 SSL。 应用 B 对同一 IP 地址使用采用新证书的基于 IP 的 SSL。 使用新证书更新应用 TLS 绑定时，失败并显示此错误，因为同一个 IP 地址已用于另一应用。 

#### <a name="solution"></a>解决方案 

若要解决此问题，请使用以下方法之一：

- 在应用中删除使用旧证书的基于 IP 的 SSL 绑定。 
- 新建使用新证书的基于 IP 的 SSL 绑定。

### <a name="you-cant-delete-a-certificate"></a>无法删除证书 

#### <a name="symptom"></a>症状

尝试删除证书时出现以下错误消息：

“无法删除证书，因为它当前正用于 TLS/SSL 绑定。 必须先删除 TLS 绑定，然后才能删除证书。”

#### <a name="cause"></a>原因

如果另一个应用使用了该证书，则可能会出现此问题。

#### <a name="solution"></a>解决方案

从应用中删除该证书的 TLS 绑定。 然后尝试删除证书。 如果仍然无法删除证书，请清除 Internet 浏览器缓存，并在新浏览器窗口中重新打开 Azure 门户。 然后尝试删除证书。

<!-- 
### You can't purchase an App Service certificate 

#### Symptom
You can't purchase an [Azure App Service certificate](./configure-ssl-certificate.md#import-an-app-service-certificate) from the Azure portal.

#### Cause and solution
This problem can occur for any of the following reasons:

- The App Service plan is Free or Shared. These pricing tiers don't support TLS. 

    **Solution**: Upgrade the App Service plan for app to Standard.

- The subscription doesn't have a valid credit card.

    **Solution**: Add a valid credit card to your subscription. 

- The subscription offer doesn't support purchasing an App Service certificate such as Microsoft Student.  

    **Solution**: Upgrade your subscription. 

- The subscription reached the limit of purchases that are allowed on a subscription.

    **Solution**: App Service certificates have a limit of 10 certificate purchases for the Pay-in-Advance and EA subscription types. For other subscription types, the limit is 3. To increase the limit, contact [Azure support](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- The App Service certificate was marked as fraud. You received the following error message: "Your certificate has been flagged for possible fraud. The request is currently under review. If the certificate does not become usable within 24 hours, contact Azure Support."

    **Solution**: If the certificate is marked as fraud and isn't resolved after 24 hours, follow these steps:

    1. Sign in to the [Azure portal](https://portal.azure.cn).
    2. Go to **App Service Certificates**, and select the certificate.
    3. Select **Certificate Configuration** > **Step 2: Verify** > **Domain Verification**. This step sends an email notice to the Azure certificate provider to resolve the problem. 
-->

## <a name="custom-domain-problems"></a>自定义域问题

### <a name="a-custom-domain-returns-a-404-error"></a>自定义域返回 404 错误 

#### <a name="symptom"></a>症状

使用自定义域名浏览到站点时，收到以下错误消息：

“错误 404 - 找不到 Web 应用”。

#### <a name="cause-and-solution"></a>原因和解决方法

**原因 1** 

配置的自定义域缺少 CNAME 或 A 记录。 

**原因 1 的解决方法**

- 如果添加了 A 记录，请确保同时添加 TXT 记录。 有关详细信息，请参阅[创建 A 记录](./app-service-web-tutorial-custom-domain.md#create-the-a-record)。
- 如果不需要对应用使用根域，我们建议使用 CNAME 记录，而不要使用 A 记录。
- 不要对同一个域同时使用 CNAME 记录和 A 记录。 此问题可能会导致冲突，并阻止域解析。 

**原因 2** 

Internet 浏览器可能仍在缓存域的旧 IP 地址。 

**原因 2 的解决方法**

清除浏览器缓存。 对于 Windows 设备，可以运行命令 `ipconfig /flushdns`。 使用 [WhatsmyDNS.net](https://www.whatsmydns.net/) 验证域是否指向应用的 IP 地址。 

### <a name="you-cant-add-a-subdomain"></a>无法添加子域 

#### <a name="symptom"></a>症状

无法将新主机名添加到应用，因此无法分配子域。

#### <a name="solution"></a>解决方案

- 咨询订阅管理员，确保有权将主机名添加到应用。
- 如果需要更多子域，我们建议将域托管服务更改为 Azure 域服务 (DNS)。 使用 Azure DNS 可将 500 个主机名添加到应用。 有关详细信息，请参阅[添加子域](https://docs.microsoft.com/archive/blogs/waws/mapping-a-custom-subdomain-to-an-azure-website)。

### <a name="dns-cant-be-resolved"></a>无法解析 DNS

#### <a name="symptom"></a>症状

收到以下错误消息：

“找不到 DNS 记录。”

#### <a name="cause"></a>原因
此问题是由以下原因之一导致的：

- 生存期 (TTL) 未过。 检查域的 DNS 配置以确定 TTL 值，然后等到期限已过。
- DNS 配置不正确。

#### <a name="solution"></a>解决方案
- 等待 48 小时，让此问题自行解决。
- 如果可以在 DNS 配置中更改 TTL 设置，请将值更改为 5 分钟，然后看看是否能解决问题。
- 使用 [WhatsmyDNS.net](https://www.whatsmydns.net/) 验证域是否指向应用的 IP 地址。 如果不是，请将 A 记录配置为应用的正确 IP 地址。

### <a name="you-need-to-restore-a-deleted-domain"></a>需要还原已删除的域 

#### <a name="symptom"></a>症状
域不再显示在 Azure 门户中。

#### <a name="cause"></a>原因 
订阅所有者可能意外删除了该域。

#### <a name="solution"></a>解决方案
如果域的删除时间不超过七天，则尚未对该域启动删除过程。 在这种情况下，可以在 Azure 门户中的同一个订阅下再次购买同一个域。 （请务必在搜索框中键入确切的域名。）此域不会重复产生费用。 如果该域的删除时间超过七天，请求助 [Azure 支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)来还原该域。

## <a name="domain-problems"></a>域问题

### <a name="you-purchased-a-tlsssl-certificate-for-the-wrong-domain"></a>为错误的域购买了 TLS/SSL 证书

#### <a name="symptom"></a>症状

为错误的域购买了应用服务证书， 并且无法将该证书更新为使用正确的域。

#### <a name="solution"></a>解决方案

删除该证书，然后购买新证书。

如果使用错误域的当前证书处于“已颁发”状态，则该证书也会产生费用。 应用服务证书不可退款，但你可以联系 [Azure 支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)，看看是否还有其他解决办法。 

### <a name="an-app-service-certificate-was-renewed-but-the-app-shows-the-old-certificate"></a>应用服务证书已续订，但应用显示旧证书 

#### <a name="symptom"></a>症状

应用服务证书已续订，但使用应用服务证书的应用仍在使用旧证书。 此外，出现需要 HTTPS 协议的警告。

#### <a name="cause"></a>原因 
应用服务会在 48 小时内自动同步证书。 在轮换或更新证书时，应用程序有时仍会检索旧证书而不是最近更新的证书， 原因是同步证书资源的作业尚未运行。 单击“同步”。同步操作会自动更新应用服务中证书的主机名绑定，而不会导致应用停机。
 
#### <a name="solution"></a>解决方案

可以强制同步证书：

1. 登录到 [Azure 门户](https://portal.azure.cn)。 选择“应用服务证书”，然后选择该证书。
2. 选择“重新生成密钥并同步”，然后选择“同步”。 同步过程需要一段时间才能完成。 
3. 同步完成后，会看到以下通知：“已成功使用最新的证书更新了所有资源。”

### <a name="domain-verification-is-not-working"></a>域验证无法进行 

#### <a name="symptom"></a>症状 
应用服务证书要求先经过域验证，然后该证书才可供使用。 选择“验证”时，验证过程失败。

#### <a name="solution"></a>解决方案
通过添加 TXT 记录来手动验证域：
 
1.  转到托管域名的域名服务 (DNS) 提供商站点。
2.  添加域的 TXT 记录，该记录使用 Azure 门户中显示的域令牌的值。 

等待几分钟以运行 DNS 传播，然后选择“刷新”按钮以触发验证。 

另一种做法是使用 HTML 网页方法来手动验证域。 此方法可让证书颁发机构确认为其颁发证书的域的域所有权。

1.  创建名为 {域验证令牌}.html 的 HTML 文件。 此文件的内容应为域验证令牌的值。
3.  将此文件上传到托管域的 Web 服务器的根目录。
4.  选择“刷新”，检查证书状态。 验证可能需要几分钟才能完成。

例如，如果为 azure.com 购买了域验证令牌为 1234abcd 的标准证书，则对 https://azure.com/1234abcd.html 发出的 Web 请求应返回 1234abcd。 

> [!IMPORTANT]
> 下达证书订单后，只有 15 天时间用于完成域验证操作。 15 天过后，证书颁发机构拒绝证书，但该证书不会产生费用。 在此情况下，请删除该证书并重试。
>
> 

### <a name="you-cant-purchase-a-domain"></a>无法购买某个域

#### <a name="symptom"></a>症状
不能在 Azure 门户中购买应用服务域。

#### <a name="cause-and-solution"></a>原因和解决方法

此问题是由以下原因之一导致的：

- Azure 订阅中没有信用卡，或信用卡无效。

    **解决方案**：将有效的信用卡添加到订阅。

- 你不是订阅所有者，因此无权购买域。

    **解决方案**：向帐户[分配“所有者”角色](../role-based-access-control/role-assignments-portal.md)。 或者联系订阅管理员以获取购买域的权限。
- 已达到订阅中可购买域数的限制。 当前限制为 20 个。

    **解决方案**：若要请求提高限制，请联系 [Azure 支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
- Azure 订阅类型不支持购买应用服务域。

    **解决方案**：将 Azure 订阅升级到其他订阅类型，例如预付费订阅。

### <a name="you-cant-add-a-host-name-to-an-app"></a>无法将主机名添加到应用 

#### <a name="symptom"></a>症状

在添加主机名的过程中无法验证域。

#### <a name="cause"></a>原因 

此问题是由以下原因之一导致的：

- 无权添加主机名。

    **解决方案**：让订阅管理员为你授予添加主机名的权限。
- 无法验证域所有权。

    **解决方案**：验证是否已正确配置 CNAME 或 A 记录。 若要将自定义域映射到应用，请创建 CNAME 记录或 A 记录。 若要使用根域，必须使用 A 记录和 TXT 记录：

    |记录类型|主机|指向|
    |------|------|-----|
    |A|@|应用的 IP 地址|
    |TXT|@|<app-name>.chinacloudsites.cn|
    |CNAME|www|<app-name>.chinacloudsites.cn|

## <a name="faq"></a>常见问题

**购买网站的自定义域后是否必须配置该域？**

通过 Azure 门户购买某个域时，应用服务应用程序会自动配置为使用该自定义域。 你不需要执行任何额外的步骤。

**是否可以使用在 Azure 门户中购买的域来指向 Azure VM？**

是的，可将该域指向 VM。 有关详细信息，请参阅[使用 Azure DNS 为 Azure 服务提供自定义域设置](../dns/dns-custom-domain.md)。

**我的域是由 GoDaddy 还是 Azure DNS 托管？**

应用服务域使用 GoDaddy 进行域注册，使用 Azure DNS 来托管域。 

**我已启用自动续订，但仍收到了有关域续订的电子邮件通知。**

如果你已启用自动续订，则不需要执行任何操作。 电子邮件通知旨在告诉你该域即将过期，如果未启用自动续订，则需要手动续订。

**在 Azure DNS 中托管域是否要付费？**

最初的域购买费用仅适用于域注册。 除了注册费用以外，Azure DNS 还会根据用量收费。 有关详细详细，请参阅 [Azure DNS 定价](https://www.azure.cn/pricing/details/dns/)。

**我的域是之前在 Azure 门户中购买的，现在想要从 GoDaddy 托管改为 Azure DNS 托管。该如何处理？**

不一定非要迁移到 Azure DNS 托管。 如果你确实想要迁移到 Azure DNS，Azure 门户中的域管理体验会提供有关转移到 Azure DNS 的步骤信息。 如果域通过应用服务购买的，则从 GoDaddy 托管迁移到 Azure DNS 的过程相对较为顺畅。

**如果通过应用服务域购买域，是否可以在 GoDaddy 而不是 Azure DNS 中托管该域？**

从 2017 年 7 月 24 日开始，在门户中购买的应用服务域将托管在 Azure DNS 中。 如果你想要使用其他托管提供商，则必须转到其网站以获取域托管解决方案。

**是否需要支付域的隐私保护费用？**

通过 Azure 门户购买域时，可以选择免费添加隐私保护。 这是通过 Azure 应用服务购买域所能获得的权益之一。

**如果我不再想要使用我的域，是否可以获得退款？**

购买域时，你可以免费试用 5 天，在此期间，可以决定是否要继续使用。 如果在这五天内你决定不需要该域，则不会产生费用。 （.uk 域例外。 购买 .uk 域会立即产生费用，而不能获得退款）。

**是否可以在订阅中的另一个 Azure 应用服务应用中使用域？**

是的。 在 Azure 门户中访问“自定义域和 TLS”边栏选项卡时，会看到已购买的域。 可将应用配置为使用其中的任何域。

**是否可将域从一个订阅转移到另一个订阅？**

可以使用 [Move-AzResource](https://docs.microsoft.com/powershell/module/az.Resources/Move-azResource) PowerShell cmdlet 将域转移到另一个订阅/资源组。

**如果我当前没有 Azure 应用服务应用，该如何管理自定义域？**

即使没有应用服务 Web 应用，也可以管理域。 域可用于虚拟机、存储等 Azure 服务。如果你打算将域用于应用服务 Web 应用，则需要添加一个未包含在免费应用服务计划中的 Web 应用才能将域绑定到 Web 应用。

**是否可将使用自定义域的 Web 应用移到另一个订阅，或者将其从应用服务环境 v1 移到 v2？**

是的，可以在订阅之间移动 Web 应用。 请遵照[如何在 Azure 中移动资源](../azure-resource-manager/management/move-resource-group-and-subscription.md)中的指导操作。 移动 Web 应用时存在一些限制。 有关详细信息，请参阅[移动应用服务资源时存在的限制](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md)。

移动 Web 应用之后，自定义域设置中的域的主机名绑定应保持不变。 无需执行额外的步骤即可配置主机名绑定。
