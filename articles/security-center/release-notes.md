---
title: Azure 安全中心的发行说明
description: 介绍 Azure 安全中心的新增功能和已更改的功能。
services: security-center
documentationcenter: na
author: Johnnytechn
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/14/2020
ms.author: v-johya
ms.openlocfilehash: 9c690e70c6a7f72ced08468fd0073c3e40741439
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402412"
---
# <a name="whats-new-in-azure-security-center"></a>Azure 安全中心的新增功能

Azure 安全中心正在积极开发中，并不断得到改进。 为及时了解最新开发成果，可在本页面查看下列相关信息：

- 新增功能
- Bug 修复
- 已弃用的功能

本页面会定期更新，请经常回来查看。 如果要查找 6 个月之前的项目，可查看 [Azure 安全中心的新增功能存档](release-notes-archive.md)。


## <a name="september-2020"></a>2020 年 9 月

9 月的更新包括以下内容：

- [连续导出中现提供漏洞评估结果](#vulnerability-assessment-findings-are-now-available-in-continuous-export)
- [优化了网络安全组建议](#network-security-group-recommendations-improved)
- [弃用了预览 AKS 建议“应在 Kubernetes 服务上定义 Pod 安全策略”](#deprecated-preview-aks-recommendation-pod-security-policies-should-be-defined-on-kubernetes-services)
- [优化了 Azure 安全中心内的电子邮件通知](#email-notifications-from-azure-security-center-improved)
- [安全功能分数不包括预览建议](#secure-score-doesnt-include-preview-recommendations)
- [建议现包含严重性指示器和刷新时间间隔](#recommendations-now-include-a-severity-indicator-and-the-freshness-interval)

### <a name="vulnerability-assessment-findings-are-now-available-in-continuous-export"></a>连续导出中现提供漏洞评估结果

使用连续导出将警报和建议实时流式传输到 Azure 事件中心、Log Analytics 工作区或 Azure Monitor。 在那里，你可以将此数据与 SIEM（例如 Power BI、Azure 数据资源管理器等）集成。

安全中心的集成漏洞评估工具返回资源结果，并将其作为“父”建议中的可操作建议（例如“应修正虚拟机中的漏洞”）。 

现在选择建议并启用“包括安全性结果”选项时，可以通过连续导出来导出安全性结果。

:::image type="content" source="./media/continuous-export/include-security-findings-toggle.png" alt-text="在连续导出配置中包括安全结果开关" :::

相关页面：

- [连续导出](continuous-export.md)

<!--Not available in MC: ### Prevent security misconfigurations by enforcing recommendations when creating new resources-->
###  <a name="network-security-group-recommendations-improved"></a>优化了网络安全组建议

以下与网络安全组相关的安全建议已得到优化，可减少误报。

- 应在与 VM 关联的 NSG 上限制所有网络端口
- 应关闭虚拟机上的管理端口
- 面向 Internet 的虚拟机应使用网络安全组进行保护
- 子网应与网络安全组关联


### <a name="deprecated-preview-aks-recommendation-pod-security-policies-should-be-defined-on-kubernetes-services"></a>弃用了预览 AKS 建议“应在 Kubernetes 服务上定义 Pod 安全策略”

如 [Azure Kubernetes 服务](/aks/use-pod-security-policies)文档中所述，弃用了预览建议“应在 Kubernetes Services 上定义 Pod 安全策略”。

Pod 安全策略（预览）功能已设置为弃用，在 2020 年 10 月 15 日之后将不再提供，以支持适用于 AKS 的 Azure Policy。

弃用 Pod 安全策略（预览版）之后，必须在使用已弃用功能的任何现有群集上禁用该功能，以执行将来的群集升级并保留在 Azure 支持范围内。


### <a name="email-notifications-from-azure-security-center-improved"></a>优化了 Azure 安全中心内的电子邮件通知

电子邮件中与安全警报相关的以下部分已得到优化： 

- 添加了发送针对所有严重性级别的电子邮件警报通知的功能
- 添加了通知在订阅中具有不同 RBAC 角色的用户的功能
- 默认情况下，我们会主动向订阅所有者通知高严重性警报（这些警报很可能表示真正的漏洞）
- 我们已从电子邮件通知配置页面中删除了电话号码字段

有关详细信息，请参阅[设置安全警报的电子邮件通知](security-center-provide-security-contact-details.md)。


### <a name="secure-score-doesnt-include-preview-recommendations"></a>安全功能分数不包括预览建议 

安全中心会持续评估资源、订阅和组织的安全问题。 然后，它将所有调查结果汇总成一个分数，让你可以一目了然地了解当前的安全状况：分数越高，识别出的风险级别就越低。

发现新的威胁后，安全中心会通过提出新的建议来提供新的安全建议。 为避免安全功能分数出现意外变化，以及为了提供一个宽限期（可以在新建议影响分数之前在此宽限期内了解新建议），安全功能分数的计算中将不再包括标记为“预览”的建议。 但仍应尽可能按这些建议进行修正，这样在预览期结束时，它们会有助于提升分数。

此外，“预览”建议不会使资源“运行不正常”。

预览建议示例如下：

:::image type="content" source="./media/secure-score-security-controls/example-of-preview-recommendation.png" alt-text="在连续导出配置中包括安全结果开关":::

[详细了解安全功能分数](secure-score-security-controls.md)。


### <a name="recommendations-now-include-a-severity-indicator-and-the-freshness-interval"></a>建议现包含严重性指示器和刷新时间间隔

现在，建议的详细信息页面包括一个刷新时间间隔指示器（如相关），并且清楚显示了建议的严重性。

:::image type="content" source="./media/release-notes/recommendations-severity-freshness-indicators.png" alt-text="在连续导出配置中包括安全结果开关":::



## <a name="august-2020"></a>2020 年 8 月

8 月的更新包括以下内容：

- [添加了对 Azure Active Directory 安全默认值（适用于多重身份验证）的支持](#added-support-for-azure-active-directory-security-defaults-for-multi-factor-authentication)
- [添加了服务主体建议](#service-principals-recommendation-added)
- [VM 漏洞评估 - 合并了建议和策略](#vulnerability-assessment-on-vms---recommendations-and-policies-consolidated)


<!--Not available in MC: ### Asset inventory - powerful new view of the security posture of your assets-->
### <a name="added-support-for-azure-active-directory-security-defaults-for-multi-factor-authentication"></a>添加了对 Azure Active Directory 安全默认值（适用于多重身份验证）的支持

安全中心已添加对[安全默认值](/active-directory/fundamentals/concept-fundamentals-security-defaults)（Microsoft 的免费标识安全保护）的全部支持。

安全默认值提供了预配置的标识安全设置，以保护组织免受与标识相关的常见攻击。 安全默认值总计已保护了逾 500 万名租户；50,000 名租户也受安全中心的保护。

现在，安全中心在识别出未启用安全默认值的 Azure 订阅时将提供安全建议。 到目前为止，安全中心建议使用条件访问启用多重身份验证，这是 Azure Active Directory (AD) 高级许可证的一部分。 对于使用 Azure AD Free 的客户，我们现在建议启用安全默认值。 

我们旨在鼓励更多客户使用 MFA 保护其云环境，并缓解对[安全功能分数](https://docs.azure.cn/security-center/secure-score-security-controls)影响最大的最高风险。

详细了解[安全默认值](/active-directory/fundamentals/concept-fundamentals-security-defaults)。


### <a name="service-principals-recommendation-added"></a>添加了服务主体建议

添加了一条新建议，该建议推荐使用管理证书来管理订阅的安全中心客户改用服务主体。

“应使用服务主体而不是管理证书来保护订阅”这一建议推荐使用服务主体或 Azure 资源管理器，以更安全地管理订阅。 

详细了解 [Azure Active Directory 中的应用程序对象和服务主体对象](/active-directory/develop/app-objects-and-service-principals#service-principal-object)。


### <a name="vulnerability-assessment-on-vms---recommendations-and-policies-consolidated"></a>VM 漏洞评估 - 合并了建议和策略

安全中心检查 VM，以检测其是否正在运行漏洞评估解决方案。 如果未找到漏洞评估解决方案，安全中心将建议简化部署。

如果发现漏洞，安全中心将建议总结结果，以便必要时进行调查和修正。

无论用户使用哪些类型的扫描仪，为确保所有用户享受一致的体验，我们已将四条建议统一为以下两条：

|统一的建议|更改描述|
|----|:----|
|**应在虚拟机上启用漏洞评估解决方案**|替换以下两条建议：<br> • 在虚拟机上启用内置漏洞评估解决方案（由 Qualys 提供技术支持）（现已弃用）（仅标准层显示此建议）<br> • 漏洞评估解决方案应安装在虚拟机上（现已弃用）（标准和免费层显示此建议）|
|**应修正虚拟机中的漏洞**|替换以下两条建议：<br>• 修正虚拟机上发现的漏洞（由 Qualys 提供支持）（现已弃用）<br>• 应通过漏洞评估解决方案修正漏洞（现已弃用）|
|||

现在可根据同一建议从 Qualys 或 Rapid7 等合作伙伴部署安全中心的漏洞评估扩展或专用许可解决方案（“BYOL”）。

此外，发现漏洞并报告到安全中心时，无论哪个漏洞评估解决方案标识了结果，都会有一条建议提醒你注意这些结果。

#### <a name="updating-dependencies"></a>更新依赖项

如果脚本、查询或自动化引用了先前的建议或策略密钥/名称，请使用下表更新引用：

##### <a name="before-august-2020"></a>2020 年 8 月之前

|建议|范围|
|----|:----|
|**在虚拟机上启用内置漏洞评估解决方案（由 Qualys 提供支持）**<br>注册表项：550e890b-e652-4d22-8274-60b3bdb24c63|内置|
|修正虚拟机上发现的漏洞（由 Qualys 提供支持）<br>注册表项：1195afff-c881-495e-9bc5-1486211ae03f|内置|
|应在虚拟机上安装漏洞评估解决方案<br>注册表项：01b1ed4c-b733-4fee-b145-f23236e70cf3|BYOL|
|**应通过漏洞评估解决方案修复漏洞**<br>注册表项：71992a2a-d168-42e0-b10e-6b45fa2ecddb|BYOL|
||||


|策略|范围|
|----|:----|
|**应对虚拟机启用漏洞评估**<br>策略 ID：501541f7-f7e7-4cd6-868c-4190fdad3ac9|内置|
|**应通过漏洞评估解决方案修正漏洞**<br>策略 ID：760a85ff-6162-42b3-8d70-698e268f648c|BYOL|
||||


##### <a name="from-august-2020"></a>2020 年 8 月之后

|建议|范围|
|----|:----|
|**应在虚拟机上启用漏洞评估解决方案**<br>密钥：ffff0522-1e88-47fc-8382-2a80ba848f5d|内置 + BYOL|
|**应修正虚拟机中的漏洞**<br>注册表项：1195afff-c881-495e-9bc5-1486211ae03f|内置 + BYOL|
||||

|策略|范围|
|----|:----|
|[应对虚拟机启用漏洞评估](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f501541f7-f7e7-4cd6-868c-4190fdad3ac9)<br>策略 ID：501541f7-f7e7-4cd6-868c-4190fdad3ac9 |内置 + BYOL|
||||


### <a name="new-aks-security-policies-added-to-asc_default-initiative---for-use-by-private-preview-customers-only"></a>新的 AKS 安全策略已添加到 ASC_default 计划 - 仅供个人预览版客户使用

为确保默认情况下保护 Kubernetes 工作负载，安全中心正在添加 Kubernetes 级别的策略和强化建议，包括带有 Kubernetes 准入控制的强制执行选项。

早期阶段的此项目包括个人预览版，并增加了 ASC_default 计划的新策略（默认情况下禁用）。

可以放心地忽略这些策略，这些策略不会对环境造成影响。 如果要启用他们，请在 https://aka.ms/SecurityPrP 中注册预览版，然后在以下选项中进行选择：

1. **单一预览版** - 仅加入此个人预览版。 明确提及“ASC 连续扫描”作为要加入的预览版。
1. **正在进行的计划** - 将添加到此个人预览版和后续个人预览版。 你将需要填写个人资料和隐私协议。


## <a name="july-2020"></a>2020 年 7 月

7 月的更新包括以下内容：
- [Azure 存储的威胁防护已扩展到包括 Azure文件存储和 Azure Data Lake Storage Gen2（预览版）](#threat-protection-for-azure-storage-expanded-to-include-azure-files-and-azure-data-lake-storage-gen2-preview)
- [启用威胁防护功能的八条新建议](#eight-new-recommendations-to-enable-threat-protection-features)
- [容器安全性优化 - 注册表扫描和刷新文档速度更快](#container-security-improvements---faster-registry-scanning-and-refreshed-documentation)
- [更新了自适应应用程序控制，添加了新建议以及对路径规则中的通配符的支持](#adaptive-application-controls-updated-with-a-new-recommendation-and-support-for-wildcards-in-path-rules)
- [已弃用六个 SQL 高级数据安全性策略](#six-policies-for-sql-advanced-data-security-deprecated)




<!--Not available in MC: ### Vulnerability assessment for virtual machines is now available for non-marketplace images-->
### <a name="threat-protection-for-azure-storage-expanded-to-include-azure-files-and-azure-data-lake-storage-gen2-preview"></a>Azure 存储的威胁防护已扩展到包括 Azure文件存储和 Azure Data Lake Storage Gen2（预览版）

Azure 存储的威胁防护可检测 Azure 存储帐户上的潜在有害活动。 安全中心在检测到对存储帐户的访问或攻击尝试时会显示警报。 

无论数据是以 blob 容器、文件共享还是以数据湖形式存储，都可以得到保护。 




### <a name="eight-new-recommendations-to-enable-threat-protection-features"></a>启用威胁防护功能的八条新建议

添加了八条新建议，简化了为以下资源类型启用 Azure 安全中心的威胁防护功能的过程：虚拟机、应用服务计划、Azure SQL 数据库服务器、计算机上的 SQL 服务器、Azure 存储帐户、Azure Kubernetes 服务群集、Azure 容器注册表注册表和 Azure Key Vault 保管库。

新建议如下所示：

- **应在 Azure SQL 数据库服务器上启用高级数据安全**
- **应在计算机的 SQL 服务器上启用高级数据安全**
- **应在 Azure 应用服务计划上启用高级威胁防护**
- **应对 Azure 容器注册表的注册表启用高级威胁防护**
- **应对 Azure Key Vault 的保管库启用高级威胁防护**
- **应对 Azure Kubernetes 服务的群集启用高级威胁防护**
- **应对 Azure 存储帐户启用高级威胁防护**
- 应对虚拟机启用高级威胁防护

这些新建议属于“启用高级威胁防护”安全控制。

建议还包括快速修复功能。 

> [!IMPORTANT]
> 修正任一建议都将产生相关资源的保护费用。 如果当前订阅中有相关资源，则立即开始计费。 或者以后在你添加资源时，开始计费。
> 
> 例如，如果订阅中没有任何 Azure Kubernetes 服务群集，并且启用了威胁防护，则不会产生任何费用。 如果以后在同一订阅中添加了群集，它将自动受到保护，并从该时间点开始计费。

有关上述各项的详细信息，请参阅[安全建议参考页面](recommendations-reference.md)。

详细了解 [Azure 安全中心的威胁防护](https://docs.azure.cn/security-center/threat-protection)。




### <a name="container-security-improvements---faster-registry-scanning-and-refreshed-documentation"></a>容器安全性优化 - 注册表扫描和刷新文档速度更快

作为对容器安全领域的持续投资的一部分，我们很高兴分享安全中心对 Azure 容器注册表中存储的容器映像的动态扫描方面的显著性能优化。 目前，扫描通常会在大约两分钟内完成。 在某些情况下，可能最多需要 15 分钟。

为更好地说明和指导 Azure 安全中心的容器安全功能，我们还更新了容器安全文档页面。 

若要详细了解安全中心的容器安全，请参阅以下文章：

- [安全中心的容器安全功能概述](https://docs.azure.cn/security-center/container-security)
- [与 Azure 容器注册表集成的详细信息](https://docs.azure.cn/security-center/azure-container-registry-integration)
- [与 Azure Kubernetes 服务集成的详细信息](https://docs.azure.cn/security-center/azure-kubernetes-service-integration)
- [扫描注册表并强化 Docker 主机的操作说明](https://docs.azure.cn/security-center/monitor-container-security)
- [威胁防护功能中适用于 Azure Kubernetes 服务群集的安全警报](https://docs.azure.cn/security-center/alerts-reference#alerts-akscluster)
- [威胁防护功能中适用于 Azure Kubernetes 服务主机的安全警报](https://docs.azure.cn/security-center/alerts-reference#alerts-containerhost)
- [容器的安全建议](https://docs.azure.cn/security-center/recommendations-reference#recs-containers)



### <a name="adaptive-application-controls-updated-with-a-new-recommendation-and-support-for-wildcards-in-path-rules"></a>更新了自适应应用程序控制，添加了新建议以及对路径规则中的通配符的支持

自适应应用程序控制功能已收到两个重要更新：

* 一项新的建议指出以前不允许的可能合法的行为。 “应更新自适应应用程序控制策略中的允许列表规则”这一新建议提示向现有策略添加新规则，以减少自适应应用程序控制违规警报的误报数。

* 路径规则现支持通配符。 在此更新中，可以使用通配符配置允许的路径规则。 支持以下两种方案：

    * 在路径末尾使用通配符允许该文件夹和子文件夹中的所有可执行文件

    * 在路径中间使用通配符来启用具有更改的文件夹名称的已知可执行文件名称（例如，包含已知可执行文件的个人用户文件夹，自动生成的文件夹名称等）。


[详细了解自适应应用程序控制](security-center-adaptive-application.md)。



### <a name="six-policies-for-sql-advanced-data-security-deprecated"></a>已弃用六个 SQL 高级数据安全性策略

即将弃用与 SQL 计算机的高级数据安全性相关的六个策略：

- 应在 SQL 托管实例的“高级数据安全”设置中将“高级威胁保护类型”设置为“全部”
- 应在 SQL Server 的高级数据安全设置中将“高级威胁防护类型”设置为“全部”
- SQL 托管实例的“高级数据安全性”设置应包含用于接收安全警报的电子邮件地址
- SQL 服务器的“高级数据安全性”设置应包含用于接收安全警报的电子邮件地址
- 应在 SQL 托管实例高级数据安全设置中启用“向管理员和订阅所有者发送电子邮件通知”
- 应在 SQL 服务器高级数据安全设置中为管理员和订阅所有者启用电子邮件通知

了解有关[内置策略](security-center-policy-definitions.md)的详细信息。





## <a name="june-2020"></a>2020 年 6 月

6 月的更新包括以下内容：
- [安全功能分数 API（预览）](#secure-score-api-preview)
- [将 Log Analytics 代理部署到 Azure Arc 计算机的两条新建议（预览）](#two-new-recommendations-to-deploy-the-log-analytics-agent-to-azure-arc-machines-preview)
- [大规模创建连续导出和工作流自动化配置的新策略](#new-policies-to-create-continuous-export-and-workflow-automation-configurations-at-scale)
- [使用 NSG 保护非面向 Internet 的虚拟机的新建议](#new-recommendation-for-using-nsgs-to-protect-non-internet-facing-virtual-machines)
- [启用威胁防护和高级数据安全性的新策略](#new-policies-for-enabling-threat-protection-and-advanced-data-security)



### <a name="secure-score-api-preview"></a>安全功能分数 API（预览）

可以通过[安全功能分数 API](https://docs.microsoft.com/rest/api/securitycenter/securescores/)（当前处于预览阶段）立即访问分数。 通过 API 方法，可灵活地查询数据，久而久之构建自己的安全功能分数报告机制。 例如，可以使用安全功能分数 API 来获取特定订阅的分数。 此外，还可以使用安全功能分数控件 API 列出安全控件和订阅的当前分数。

有关使用安全功能分数 API 实现的外部工具的示例，请参阅 [GitHub 社区的安全功能分数区域](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score)。

详细了解 [Azure 安全中心的安全功能分数和安全控制](secure-score-security-controls.md)。



### <a name="two-new-recommendations-to-deploy-the-log-analytics-agent-to-azure-arc-machines-preview"></a>将 Log Analytics 代理部署到 Azure Arc 计算机的两条新建议（预览）

添加了两条新建议，以帮助将 [Log Analytics 代理](/azure-monitor/platform/log-analytics-agent)部署到 Azure Arc 计算机，并确保其受 Azure 安全中心的保护：

- Log Analytics 代理应安装在基于 Windows 的 Azure Arc 计算机上(预览)
- Log Analytics 代理应安装在基于 Linux 的 Azure Arc 计算机上(预览)

这些新建议将出现在“应在计算机上安装监视代理”这一现有（相关）建议所在的四个安全控制中：修正安全配置、应用自适应应用程序控制、应用系统更新，以及启用 Endpoint Protection。

建议还包括快速修复功能，以帮助加快部署过程。 

有关这两项新建议的详细信息，请参阅[计算和应用建议](recommendations-reference.md#recs-computeapp)。

若要详细了解 Azure 安全中心如何使用代理，请参阅[什么是 Log Analytics 代理？](https://docs.azure.cn/security-center/faq-data-collection-agents#what-is-the-log-analytics-agent)

详细了解 [Azure Arc 计算机的扩展](https://docs.microsoft.com/azure/azure-arc/servers/manage-vm-extensions#enable-extensions-from-the-portal)。



### <a name="new-policies-to-create-continuous-export-and-workflow-automation-configurations-at-scale"></a>大规模创建连续导出和工作流自动化配置的新策略

自动执行组织的监视和事件响应流程可以显著缩短调查和缓解安全事件所需的时间。

若要在整个组织中部署自动化配置，请使用以下内置的“DeployIfdNotExist”Azure 策略来创建和配置[连续导出](continuous-export.md)和[工作流自动化](workflow-automation.md)过程：

可在 Azure 策略中找到这些策略：


|目标  |策略  |策略 ID  |
|---------|---------|---------|
|将内容连续导出到事件中心|[为 Azure 安全中心警报和建议部署“导出到事件中心”配置](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fcdfcce10-4578-4ecd-9703-530938e4abcb)|cdfcce10-4578-4ecd-9703-530938e4abcb|
|将内容连续导出到 Log Analytics 工作区|[为 Azure 安全中心警报和建议配置“导出到 Log Analytics 工作区”配置](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fffb6f416-7bd2-4488-8828-56585fef2be9)|ffb6f416-7bd2-4488-8828-56585fef2be9|
|安全警报的工作流自动化|[为 Azure 安全中心警报部署工作流自动化](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff1525828-9a90-4fcf-be48-268cdd02361e)|f1525828-9a90-4fcf-be48-268cdd02361e|
|安全建议的工作流自动化|[为 Azure 安全中心建议部署工作流自动化](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f73d6ab6c-2475-4850-afd6-43795f3492ef)|73d6ab6c-2475-4850-afd6-43795f3492ef|
||||

开始使用[工作流自动化模板](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation)。

若要详细了解如何使用这两种导出策略，请参阅[通过 Policy 连续导出 Azure 安全中心警报和建议](https://techcommunity.microsoft.com/t5/azure-security-center/continuously-export-azure-security-center-alerts-and/ba-p/1440745)。


### <a name="new-recommendation-for-using-nsgs-to-protect-non-internet-facing-virtual-machines"></a>使用 NSG 保护非面向 Internet 的虚拟机的新建议

“实现安全最佳做法”安全控制现包括以下新建议：

- **应使用网络安全组来保护非面向 Internet 的虚拟机**

“应使用网络安全组保护面向 Internet 的虚拟机”这一现有建议不区分面向 Internet 的虚拟机和面向非 Internet 的虚拟机。 对于这两种情况，如果未将 VM 分配给网络安全组，则会生成高严重性建议。 这一新建议将区分面向非 Internet 的计算机，以减少误报并避免出现不必要的高严重性警报。

有关详细详细，请参阅[网络建议](recommendations-reference.md#recs-network)表。




### <a name="new-policies-for-enabling-threat-protection-and-advanced-data-security"></a>启用威胁防护和高级数据安全性的新策略

以下新策略已添加到 ASC Default 计划，旨在帮助为相关资源类型启用威胁防护或高级数据安全性。

可在 Azure 策略中找到这些策略：


| 策略                                                                                                                                                                                                                                                                | 策略 ID                            |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| [应在 Azure SQL 数据库服务器上启用高级数据安全](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f7fe3b40f-802b-4cdd-8bd4-fd799c948cc2)     | 7fe3b40f-802b-4cdd-8bd4-fd799c948cc2 |
| [应在计算机的 SQL 服务器上启用高级数据安全](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f6581d072-105e-4418-827f-bd446d56421b) | 6581d072-105e-4418-827f-bd446d56421b |
| [应对 Azure 存储帐户启用高级威胁防护](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f308fbb08-4ab8-4e67-9b29-592e93fb94fa)           | 308fbb08-4ab8-4e67-9b29-592e93fb94fa |
| [应对 Azure Key Vault 的保管库启用高级威胁防护](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f0e6763cc-5078-4e64-889d-ff4d9a839047)           | 0e6763cc-5078-4e64-889d-ff4d9a839047 |
| [应在 Azure 应用服务计划上启用高级威胁防护](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f2913021d-f2fd-4f3d-b958-22354e2bdbcb)                | 2913021d-f2fd-4f3d-b958-22354e2bdbcb |
| [应对 Azure 容器注册表的注册表启用高级威胁防护](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fc25d9a16-bc35-4e15-a7e5-9db606bf9ed4)   | c25d9a16-bc35-4e15-a7e5-9db606bf9ed4 |
| [应对 Azure Kubernetes 服务的群集启用高级威胁防护](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f523b5cd1-3e23-492f-a539-13118b6d1e3a)   | 523b5cd1-3e23-492f-a539-13118b6d1e3a |
| [应在虚拟机上启用高级威胁防护](https://portal.azure.cn/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4da35fc9-c9e7-4960-aec9-797fe7d9051d)           | 4da35fc9-c9e7-4960-aec9-797fe7d9051d |
|                                                                                                                                                                                                                                                                       |                                      |

详细了解 [Azure 安全中心的威胁防护](https://docs.azure.cn/security-center/threat-protection)。





## <a name="may-2020"></a>2020 年 5 月

5 月的更新包括以下内容：
- [警报抑制规则（预览版）](#alert-suppression-rules-preview)
- [对实时 (JIT) 虚拟机 (VM) 访问权限的更改](#changes-to-just-in-time-jit-virtual-machine-vm-access)
- [自定义建议已移至单独的安全控件](#custom-recommendations-have-been-moved-to-a-separate-security-control)
- [已添加开关，可在控件中显示建议或以简单列表的形式显示](#toggle-added-to-view-recommendations-in-controls-or-as-a-flat-list)
- [扩展了“实现安全最佳做法”这一安全控件](#expanded-security-control-implement-security-best-practices)
- [具有自定义元数据的自定义策略现已正式发布](#custom-policies-with-custom-metadata-are-now-generally-available)
- [故障转储分析功能正在迁至无文件攻击检测中](#crash-dump-analysis-capabilities-migrating-to-fileless-attack-detection)


### <a name="alert-suppression-rules-preview"></a>警报抑制规则（预览版）

这项新功能目前为预览版，它可帮助缓解警报疲劳。 可使用规则来自动隐藏已知无害或已知与你组织中的正常活动相关的警报。 这可让你专注于最相关的威胁。 

仍将生成与你启用的抑制规则相匹配的警报，但它们的状态将设置为“已取消”。 你可在 Azure 门户中查看状态，也可在安全中心查看安全警报。

抑制规则定义了自动取消警报所应遵循的条件。 通常，使用抑制规则来：

- 取消已标识为“误报”的警报

- 取消限制过于频繁地触发而失去作用的警报

详细了解如何[取消来自 Azure 安全中心威胁防护服务的警报](alerts-suppression-rules.md)。


<!--Not available in MC: ### Virtual machine vulnerability assessment is now generally available-->
### <a name="changes-to-just-in-time-jit-virtual-machine-vm-access"></a>对实时 (JIT) 虚拟机 (VM) 访问权限的更改

安全中心包含一项可选功能，可保护 VM 的管理端口。 这可抵御最常见形式的暴力攻击。

本次更新就此功能进行了以下更改：

- 重命名了推荐你在 VM 上启用 JIT 的建议。 之前称为“应在虚拟机上应用实时网络访问控制”，而现在叫做“应通过即时网络访问控制来保护虚拟机的管理端口”。

- 建议仅在有管理端口打开时才触发。

详细了解 [JIT 访问功能](security-center-just-in-time.md)。


### <a name="custom-recommendations-have-been-moved-to-a-separate-security-control"></a>自定义建议已移至单独的安全控件

安全功能分数增强版引入的其中一个安全控制是“实现安全最佳做法”。 为订阅创建的所有自定义建议已自动放入该控件中。 

为便于查找自定义建议，我们已将这些建议移到一个名为“自定义建议”的专用安全控件中。 此控件不会影响你的安全功能分数。

要详细了解安全控件，请参阅 [Azure 安全中心的安全功能分数增强版（预览版）](secure-score-security-controls.md)。


### <a name="toggle-added-to-view-recommendations-in-controls-or-as-a-flat-list"></a>已添加开关，可在控件中显示建议或以简单列表的形式显示

安全控件是相关安全建议的逻辑组。 它们反映了易受攻击的攻击面。 控件是一组安全建议，附有帮助你实施这些建议的说明。

若要立即查看组织对每个攻击面的保护情况，请查看每个安全控件的分数。

默认情况下，你的建议显示在安全控件中。 通过本次更新，你还可以采用列表形式显示它们。 若要以简单列表的形式查看它们，且列表按受影响的资源的运行状况排序，请使用新的“按控件分组”开关。 开关位于门户中列表的上面。

安全控件及其开关是新的安全功能分数体验的一部分。 请记得在门户中提供反馈。

要详细了解安全控件，请参阅 [Azure 安全中心的安全功能分数增强版（预览版）](secure-score-security-controls.md)。

![建议的“按控制分组”开关](\media\secure-score-security-controls\recommendations-group-by-toggle.gif)

### <a name="expanded-security-control-implement-security-best-practices"></a>扩展了“实现安全最佳做法”这一安全控件 

安全功能分数增强版引入的其中一个安全控制是“实现安全最佳做法”。 如果建议在此控件中显示，则不影响安全功能分数。 

通过本次更新，已将三项建议从它们原先所在的控件移动到这个最佳做法控件中。 我们采取此步骤的原因是我们判定这三项建议的风险比最初设想的要低。

此外，还引入了两项新建议，它们也添加到了此控件中。

移动的三项建议如下：

- **应在对订阅拥有读取权限的帐户上启用 MFA**（原先位于“启用 MFA”控件中）
- **应从订阅中删除具有读取权限的外部帐户**（原先位于“管理访问和权限”控件中）
- **只多只为订阅指定 3 个所有者**（原先位于“管理访问和权限”控件中）

添加到控件中的两项新建议如下：

- **应在 Windows 虚拟机上安装来宾配置扩展（预览版）** - 如果使用 [Azure Policy 来宾配置](/governance/policy/concepts/guest-configuration)，则可在虚拟机中查看服务器和应用程序设置（仅限 Windows）。

- **应在计算机上启用 Windows Defender 攻击防护（预览版）** - Windows Defender 攻击防护采用 Azure Policy 来宾配置代理。 攻击防护服务具有 4 个组件，旨在锁定设备来阻隔各种攻击途径，并阻止恶意软件攻击中常用的行为，同时让企业能够平衡其安全风险和生产力要求（仅限 Windows）。

要详细了解 Windows Defender 攻击防护，可参阅[创建和部署攻击防护策略](https://docs.microsoft.com/mem/configmgr/protect/deploy-use/create-deploy-exploit-guard-policy)。

要详细了解安全控件，请参阅[安全功能分数增强版（预览版）](secure-score-security-controls.md)。



### <a name="custom-policies-with-custom-metadata-are-now-generally-available"></a>具有自定义元数据的自定义策略现已正式发布

自定义策略现显示在安全中心的建议体验、安全功能分数和法规符合性标准仪表板中。 此功能现已正式发布，可用于在安全中心扩大你组织的安全评估范围。 

在 Azure 策略中创建自定义计划，向该计划添加策略并将它加入 Azure 安全中心，然后将它作为建议直观呈现。

现在，我们还添加了可编辑自定义建议元数据的选项。 元数据选项中有严重级别、修正步骤和威胁信息等。  

详细了解[利用详细信息增强自定义建议](custom-security-policies.md#enhance-your-custom-recommendations-with-detailed-information)。



### <a name="crash-dump-analysis-capabilities-migrating-to-fileless-attack-detection"></a>故障转储分析功能正在迁至无文件攻击检测中 

我们正在将 Windows 故障转储分析 (CDA) 检测功能集成到[无文件攻击检测](https://docs.azure.cn/security-center/threat-protection#windows-fileless)中。 无文件攻击检测分析改进了 Windows 计算机的以下安全警报：“发现代码注入”、“检测到伪装 Windows 模块”、“发现 Shellcode”和“检测到可疑的代码段”。

该转换的一些优势如下：

- **主动及时检测恶意软件** - 使用 CDA 方法时，会等到故障发生后再运行分析来查找恶意项目。 使用无文件攻击检测后，可在内存中威胁正在运行时主动识别它们。 

- **警报信息更丰富** - 来自无文件攻击检测的安全警报包含 CDA 中不提供的丰富信息，例如有效网络连接信息。 

- **警报聚合** - CDA 在一个故障转储中检测到多个攻击模式时，会触发多个安全警报。 而无文件攻击检测将从同一进程中确定的所有攻击模式组合到一个警报中，免去了关联多个警报的必要性。

- **降低了对 Log Analytics 工作区的要求** - 包含潜在敏感数据的故障转储将无法上传到 Log Analytics 工作区。



## <a name="april-2020"></a>2020 年 4 月

4 月的更新包括以下内容：
- [标识建议现包含在 Azure 安全中心的免费层中](#identity-recommendations-now-included-in-azure-security-center-free-tier)


### <a name="identity-recommendations-now-included-in-azure-security-center-free-tier"></a>标识建议现包含在 Azure 安全中心的免费层中

Azure 安全中心免费层中针对标识和访问的安全建议现已正式发布。 这是我们努力使云安全状态管理 (CSPM) 功能免费而取得的成果之一。 截至目前，这些建议仅在标准定价层中提供。

标识和访问建议的示例包括：

- “应在对订阅拥有所有者权限的帐户上启用多重身份验证。”
- “最多只能为订阅指定 3 个所有者。”
- “应从订阅中删除弃用的帐户。”

如果你有订阅在免费定价层，则此更改将影响它们的安全功能分数，因为它们之前从未接受过标识和访问安全性评估。

详细了解[标识和访问建议](recommendations-reference.md#recs-identity)。

详细了解[监视标识和访问](security-center-identity-access.md)。

