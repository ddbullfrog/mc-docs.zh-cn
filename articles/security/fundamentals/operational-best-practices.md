---
title: 针对 Azure 资产的安全性最佳做法
titleSuffix: Azure security
description: 本文提供了用于保护 Azure 中的数据、应用程序和其他资产的一系列操作最佳做法。
services: security
documentationcenter: na
author: Johnnytechn
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/12/2020
ms.author: v-johya
origin.date: 05/06/2019
ms.openlocfilehash: 771e2006eda270c70d0505522803413ae9094868
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127720"
---
# <a name="azure-operational-security-best-practices"></a>Azure 操作安全性最佳做法
本文提供了用于保护 Azure 中的数据、应用程序和其他资产的一系列操作最佳做法。

最佳做法以观点的共识以及 Azure 平台功能和特性集为基础。 观点和技术将随着时间改变，本文会定期更新以反映这些更改。

## <a name="define-and-deploy-strong-operational-security-practices"></a>定义并部署强大的操作安全做法
Azure 操作安全性是指用户可用于在 Azure 中保护其数据、应用程序和其他资产的服务、控件和功能。 Azure 操作安全性建立在一个框架上，该框架融合了通过 Microsoft 独有的功能获得的知识，包括 [安全开发生命周期 (SDL)](https://www.microsoft.com/sdl)、[Microsoft 安全响应中心](https://www.microsoft.com/msrc?rtc=1)计划以及对网络安全威胁形态的深刻认识。

## <a name="manage-and-monitor-user-passwords"></a>管理和监视用户密码
下表列出了与管理用户密码相关的一些最佳做法：

**最佳做法** ：确保你在云中具有适当级别的密码保护。   
**详细信息** ：按照 [Microsoft 密码指南](https://www.microsoft.com/research/publication/password-guidance/)中的指南进行操作，该指南的适用范围是 Microsoft 标识平台（Azure Active Directory、Active Directory 和 Microsoft 帐户）的用户。

<!-- not available-->

- 检测影响组织标识的潜在漏洞
- 配置自动响应检测到的与组织标识相关的可疑操作
- 调查可疑事件，并采取适当的措施进行解决

## <a name="receive-incident-notifications-from-microsoft"></a>接收来自 Microsoft 的事件通知
确保你的安全运营团队接收来自 Microsoft 的 Azure 事件通知。 事件通知让你的安全团队知道你已经破坏了某个 Azure 资源，目的是让他们可以快速响应并修正潜在的安全风险。

在 Azure 注册门户中，你可以确保管理员联系信息包含用来进行安全操作通知的详细信息。 联系人详细信息为电子邮件地址和电话号码。

## <a name="organize-azure-subscriptions-into-management-groups"></a>将 Azure 订阅组织到管理组中
如果你的组织有多个订阅，则可能需要一种方法来高效地管理这些订阅的访问权限、策略和符合性。 [Azure 管理组](/governance/management-groups/create)提供了高于订阅的一个作用域级别。 可将订阅组织到名为“管理组”的容器中，并将治理条件应用到管理组。 管理组中的所有订阅都将自动继承应用于管理组的条件。

可以在目录中构建管理组和订阅的灵活结构。 为每个目录指定了一个称为根管理组的顶级管理组。 此根管理组内置在层次结构中，包含其所有下级管理组和订阅。 该根管理组允许在目录级别应用全局策略和 Azure 角色分配。

下面是管理组使用方面的一些最佳做法：

**最佳做法** ：确保在添加新订阅时，它们会应用治理元素，例如策略和权限。   
**详细信息** ：使用根管理组分配适用于所有 Azure 资产的企业范围的安全元素。 策略和权限是元素的示例。

**最佳做法** ：将顶级管理组与分段策略匹配，以便在每个段中实现控制和策略一致性。   
**详细信息** ：在根管理组下为每个段创建一个管理组。 请勿在根下创建任何其他管理组。

**最佳做法** ：限制管理组深度，以避免出现影响操作和安全性的混乱。   
**详细信息** ：将层次结构限制为三个级别（包括根在内）。

**最佳做法** ：使用根管理组，仔细选择要应用于整个企业的项。   
**详细信息** ：确保根管理组元素明确需要在每个资源中应用，并且它们的影响很小。

典型的候选项包括：

- 具有明确业务影响的法规要求（例如，与数据主权相关的限制）
- 对操作几乎没有潜在负面影响的要求，例如，其审核效果或 RBAC 权限分配已经过仔细审查的策略

**最佳做法** ：在根管理组上应用所有企业范围的更改（策略、RBAC 模型等）之前，请仔细规划并测试它们。   
**详细信息** ：根管理组中的更改可能会影响 Azure 上的每个资源。 尽管它们提供了一种强大的方法来确保整个企业中的一致性，但错误或不正确的使用可能会对生产操作产生负面影响。 请在测试实验室或生产试点中测试对根管理组的所有更改。

<!-- blueprints not available -->

## <a name="monitor-storage-services-for-unexpected-changes-in-behavior"></a>监视存储服务的意外行为更改
诊断和排查在云环境中托管的分布式应用程序中的问题可能会比在传统环境中更复杂。 应用程序可以部署在 PaaS 或 IaaS 基础结构、本地、移动设备，或这些环境的某种组合中。 应用程序的网络流量可能会遍历公用和专用网络，你的应用程序可能使用多种存储技术。

应持续监视应用程序使用的存储服务是否存在任何意外行为更改（如响应时间变长）。 若要收集更详细的数据和深度分析问题，请使用日志记录。 从监视和日志记录获取的诊断信息将有助于确定应用程序所遇到问题的根本原因。 然后，可以排查该问题，并确定修正问题的相应步骤。

[Azure 存储分析](../../storage/common/storage-analytics.md)执行日志记录并为 Azure 存储帐户提供指标数据。 建议使用此数据跟踪请求、分析使用情况趋势以及诊断存储帐户的问题。

## <a name="prevent-detect-and-respond-to-threats"></a>防范、检测和应对威胁
[Azure 安全中心](../../security-center/security-center-intro.md)增强了对 Azure 资源安全的可见性和可控性，可帮助你预防、检测和响应威胁。 它提供对 Azure 订阅的集成安全监视和策略管理，帮助检测可能被忽略的威胁，且适用于各种安全解决方案。

安全中心的免费层仅为 Azure 资源提供有限的安全性。 标准层将这些功能扩展到本地和其他云中。 借助安全中心标准层，可以查找和修复安全漏洞、应用访问控制和应用程序控制来阻止恶意活动、使用分析和智能功能检测威胁，以及在受到攻击时迅速做出响应。 可以尝试安全中心标准版，头 60 天免费。 建议[将 Azure 订阅升级到安全中心标准层](../../security-center/security-center-get-started.md)。

使用安全中心，可以通过一个集中化视图查看所有 Azure 资源的安全状态。 一眼就可验证适当的安全控件是否配置到位且配置正确，还可快速确认任何需要注意的资源。

<!-- Azure Sentinel not available-->

下面是一些用于预防、检测和响应威胁的最佳做法：

<!-- not available-->

**最佳做法** ：找到最严重的安全漏洞，以便确定调查优先级。   
**详细信息** ：查看你的 [Azure 安全评分](../../security-center/secure-score-security-controls.md)，了解 Azure 安全中心内置的 Azure 策略和计划所产生的建议。 这些建议有助于解决顶级风险，例如安全更新、终结点保护、加密、安全配置、WAF 缺失、VM 连接到 Internet 等方面的风险。

安全评分基于 Internet 安全中心 (CIS) 控件，允许你根据外部源对组织的 Azure 安全性进行基准测试。 外部验证可帮助验证并扩充团队的安全策略。

**最佳做法** ：监视计算机、网络、存储和数据服务以及应用程序的安全状况，发现潜在的安全问题并确定其优先级。  
**详细信息** ：按照安全中心的 [安全建议](../../security-center/security-center-recommendations.md)操作，并从优先级最高的项开始。

<!--cannot export -->

**最佳做法** ：将 Azure 日志与你的 SIEM 集成。   
**详细信息** ： [使用 Azure Monitor 收集和导出数据](/azure-monitor/overview#integrate-and-export-data)。 此做法对于启用安全事件调查至关重要，而在线日志保留期是有限的。<!-- Azure Sentinel not available -->

## <a name="monitor-end-to-end-scenario-based-network-monitoring"></a>监视基于端到端方案的网络监视
客户在 Azure 中通过合并虚拟网络、ExpressRoute、应用程序网关和负载均衡器等网络资源来构建端到端网络。 监视适用于每个网络资源。

[Azure 网络观察程序](../../network-watcher/network-watcher-monitoring-overview.md)是一项区域性服务。 其诊断和可视化工具可用于在网络方案级别监视和诊断 Azure 内部以及传入和传出 Azure 的流量的状态。

以下是网络监视和可用工具的最佳做法。

**最佳做法** ：使用数据包捕获实现远程网络监视的自动化。  
**详细信息** ：使用网络观察程序监视和诊断网络问题，无需登录 VM。 通过设置警报触发[数据包捕获](../../network-watcher/network-watcher-alert-triggered-packet-capture.md)，并获取数据包级别上的实时性能信息访问权限。 如果遇到问题，可进行详细调查，获得更精确的诊断。

**最佳做法** ：使用流日志深入了解网络流量。  
**详细信息** ：使用 [网络安全组流日志](../../network-watcher/network-watcher-nsg-flow-logging-overview.md)更深入地了解网络流量模式。 流日志中的信息可帮助收集符合性数据、审核和监视网络安全配置文件。

**最佳做法** ：诊断 VPN 连接问题。  
**详细信息** ：使用网络观察程序来 [诊断最常见的 VPN 网关和连接问题](../../network-watcher/network-watcher-diagnose-on-premises-connectivity.md)。 不仅可以确定问题，还可以使用详细日志进一步调查。

<!-- devops not available-->
<!-- DDoS not available -->

## <a name="enable-azure-policy"></a>启用 Azure Policy
[Azure Policy](/governance/policy/overview) 是 Azure 中的一项服务，用于创建、分配和管理策略。 这些策略将在整个资源中强制实施规则和效果，使这些资源符合公司标准和服务级别协议。 Azure Policy 通过评估资源是否符合指定策略来满足此需求。

启用 Azure Policy 来进行监视并强制实施组织的书面策略。 这样就可以集中管理混合云工作负荷中的安全策略，确保符合公司或法规安全要求。 了解如何[创建和管理策略以强制实施合规性](../../governance/policy/tutorials/create-and-manage.md)。 有关策略元素的概述，请参阅 [Azure Policy 定义结构](../../governance/policy/concepts/definition-structure.md)。

下面是在采用 Azure Policy 后要遵循的一些安全性最佳做法：

**最佳做法** ：Azure Policy 支持多种类型的效果。 可以在 [Azure Policy 定义结构](../../governance/policy/concepts/definition-structure.md#policy-rule)中了解相关信息。 **拒绝** 效果和 **修正** 效果可能会给业务运营带来负面影响，因此请从 **审核** 效果开始以限制策略带来的负面影响风险。   
**详细信息** ： [以审核模式开始策略部署](../../governance/policy/concepts/definition-structure.md#policy-rule)，然后推进到 **拒绝** 或 **修正** 。 在推进到 **拒绝** 或 **修正** 之前，请测试并查看审核效果的结果。

有关详细信息，请参阅[创建和管理策略以强制实施符合性](../../governance/policy/tutorials/create-and-manage.md)。

**最佳做法** ：确定负责监视策略违规的角色，并确保快速执行正确的修正操作。   
**详细信息** ：让已分配的角色通过 [Azure 门户](../../governance/policy/how-to/get-compliance-data.md#portal)或 [命令行](../../governance/policy/how-to/get-compliance-data.md#command-line)来监视符合性。

**最佳做法** ：Azure Policy 是组织的书面策略的技术表示形式。 将所有 Azure 策略定义映射到组织策略，以减少混乱并增强一致性。   
**详细信息** ：通过在[策略定义](../../governance/policy/concepts/definition-structure.md#display-name-and-description)或[计划定义](../../governance/policy/concepts/initiative-definition-structure.md#metadata)说明中添加对组织策略的引用，在组织的文档中或 Azure Policy 定义本身中记录映射。


<!-- not available -->

## <a name="next-steps"></a>后续步骤
有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](best-practices-and-patterns.md)。

以下资源提供了有关 Azure 安全性及相关 Microsoft 服务的更多常规信息：
* [Azure 安全团队博客](https://blogs.msdn.microsoft.com/azuresecurity/) - 随时掌握 Azure 安全性的最新信息
* [Microsoft 安全响应中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中报告 Microsoft 安全漏洞（包括 Azure 问题）或将其通过电子邮件发送到 secure@microsoft.com

