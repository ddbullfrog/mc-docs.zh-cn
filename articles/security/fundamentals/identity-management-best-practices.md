---
title: Azure 标识和访问安全最佳做法 | Azure Docs
description: 本文提供一系列有关使用内置 Azure 功能进行标识管理和访问控制的最佳实践。
services: security
documentationcenter: na
author: Johnnytechn
manager: RKarlin
editor: TomSh
ms.assetid: 07d8e8a8-47e8-447c-9c06-3a88d2713bc1
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/12/2020
ms.author: v-johya
origin.date: 10/28/2019
ms.openlocfilehash: ffcf6b282c26d4a9096f6bcf257e0fef3281999d
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128303"
---
# <a name="azure-identity-management-and-access-control-security-best-practices"></a>Azure 标识管理和访问控制安全最佳实践

本文介绍一系列 Azure 标识管理和访问控制安全最佳实践。 这些最佳做法衍生自我们的 [Azure AD](../../active-directory/fundamentals/active-directory-whatis.md) 经验和客户经验。

对于每项最佳做法，本文将说明：

* 最佳实践是什么
* 为何要启用该最佳实践
* 如果无法启用该最佳实践，可能的结果是什么
* 最佳实践的可能替代方案
* 如何学习启用最佳实践

这篇 Azure 标识管理和访问控制安全最佳实践以共识以及 Azure 平台功能和特性集（因为在编写本文时已存在）为基础。

<!-- not available -->

看法和技术将随着时间改变，本文会定期更新以反映这些更改。

本文中介绍的 Azure 标识管理和访问控制安全最佳实践包括：

* 将标识视为主要安全边界
* 集中化标识管理
* 管理连接的租户
* 规划例行的安全改进
* 启用密码管理
* 对用户强制执行多重身份验证
* 使用基于角色的访问控制
* 控制资源所在的位置
* 使用 Azure AD 进行存储身份验证

## <a name="treat-identity-as-the-primary-security-perimeter"></a>将标识视为主要安全边界

许多人认为标识是主要安全边界。 这与以网络安全为重点的传统做法不同。 网络边界出现越来越多的漏洞，与 [BYOD](https://aka.ms/byodcg) 设备和云应用程序激增之前相比，边界防御不再那样有效。

[Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) 是用于标识和访问管理的 Azure 解决方案。 Azure AD 是 Microsoft 提供的多租户、基于云的目录和标识管理服务。 它将核心目录服务、应用程序访问管理和标识保护融入一个解决方案中。

以下部分列出了使用 Azure AD 实现标识和访问安全性的最佳做法。

**最佳做法** ：围绕用户和服务标识进行安全控制和检测。
**详细信息** ：使用 Azure AD 并置控制和标识。

## <a name="centralize-identity-management"></a>集中化标识管理

在[混合标识](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?)方案中，我们建议集成本地目录和云目录。 通过集成，IT 团队可以在一个位置集中管理帐户，而不管帐户是在哪里创建的。 集成还通过提供用于访问云和本地资源的通用标识，从而帮助用户提高工作效率。

**最佳做法** ：建立一个 Azure AD 实例。 一致性和一个权威源不仅会提高简明性，还会减少人为错误和配置复杂性带来的安全风险。
**详细信息** ：指定一个 Azure AD 目录作为企业帐户和组织帐户的权威源。

**最佳做法** ：将本地目录与 Azure AD 进行集成。  
**详细信息** ：使用 [Azure AD Connect](/active-directory/connect/active-directory-aadconnect) 将本地目录与云目录同步。

> [!Note]
> 存在[影响 Azure AD Connect 性能的因素](../../active-directory/hybrid/plan-connect-performance-factors.md)。 确保 Azure AD Connect 有足够的容量来防止性能不佳的系统影响安全性和工作效率。 大型或复杂的组织（预配超过 10 万个对象的组织）应遵循[建议](../../active-directory/hybrid/whatis-hybrid-identity.md)来优化其 Azure AD Connect 实现。

**最佳做法** ：不要将现有 Active Directory 实例中权限高的帐户同步到 Azure AD。
**详细信息** ：请勿更改已筛选掉这些帐户的默认 [Azure AD Connect 配置](../../active-directory/hybrid/how-to-connect-sync-configure-filtering.md)。 此配置降低了攻击者从云透视到本地资源（这可能会造成重大事件）的风险。

**最佳做法** ：启用密码哈希同步。  
**详细信息** ：密码哈希同步是用于将用户密码哈希从本地 Active Directory 实例同步到基于云的 Azure AD 实例的功能。 此同步有助于防止重放先前攻击中泄露的凭据。

即使决定使用 Active Directory 联合身份验证服务 (AD FS) 或其他标识提供者进行联合身份验证，也可以选择性地设置密码哈希同步作为备用机制，以应对本地服务器发生故障或临时不可用的情况。 借助此同步，用户可以使用与登录本地 Active Directory 实例相同的密码来登录服务。 如果用户对其他未连接到 Azure AD 的服务使用过相同的电子邮件地址和密码，此同步还可便于标识保护将同步的密码哈希与已知被盗用的密码进行比较，从而检测被盗用的凭据。

有关详细信息，请参阅[使用 Azure AD Connect 同步实现密码哈希同步](/active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization)。

<!-- B2B2 B2C not availavle  -->

未将其本地标识与云标识集成的组织在管理帐户方面可能开销更大。 这种开销增加了出错和安全漏洞的可能性。

> [!Note]
> 你需要选择关键帐户将驻留在哪些目录中，以及所使用的管理工作站是由新的云服务托管，还是由现有进程托管。 使用现有的管理和标识预配流程可以降低一些风险，但也可能会造成攻击者入侵本地帐户并转向云的风险。 不妨对不同的角色（例如，IT 管理员与业务部门管理员）使用不同的策略。 可以使用两个选项。 第一个选项是创建不与本地 Active Directory 实例同步的 Azure AD 帐户。 将你的管理工作站加入到 Azure AD，你可以使用 Microsoft Intune 对其进行管理和修补。 第二种选择是，通过同步到本地 Active Directory 实例来使用现有的管理员帐户。 使用 Active Directory 域中的现有工作站来实现管理和安全性。

## <a name="manage-connected-tenants"></a>管理已连接的租户
你的安全组织需要能够查看订阅来评估风险，并确定是否遵循了组织的策略和任何法规要求。 你应确保安全组织能够查看所有（通过 [Azure ExpressRoute](../../expressroute/expressroute-introduction.md) 或[站点到站点 VPN](../../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)）连接到生产环境和网络的订阅。 Azure AD 中的[全局管理员/公司管理员](../../active-directory/users-groups-roles/directory-assign-admin-roles.md#company-administrator-permissions)可以将自己的访问权限提升为[用户访问管理员](../../role-based-access-control/built-in-roles.md#user-access-administrator)角色，并查看所有连接到环境的订阅和管理组。

请参阅[提升访问权限以管理所有 Azure 订阅和管理组](../../role-based-access-control/elevate-access-global-admin.md)，以确保你和你的安全组可以查看连接到你的环境的所有订阅或管理组。 评估风险后，应删除此提升的访问权限。

<!--not available -->

## <a name="turn-on-conditional-access"></a>启用条件访问

用户可能会从任意位置使用各种设备和应用访问组织的资源。 作为一名 IT 管理员，你需要确保这些设备符合安全性和符合性标准。 仅关注谁可以访问资源不再能满足需求。

为了平衡安全性与工作效率，在做出访问控制决策之前，需要考虑如何访问资源。 使用 Azure AD 条件访问，可以满足这一需求。 使用条件访问，可以根据访问云应用的条件做出自动访问控制决策。

**最佳做法** ：管理和控制对公司资源的访问。  
**详细信息** ：根据 SaaS 应用和 Azure AD 连接的应用的组、位置和应用敏感度，配置通用 Azure AD [条件访问策略](../../active-directory/conditional-access/concept-conditional-access-policy-common.md)。

## <a name="plan-for-routine-security-improvements"></a>计划例程安全改进

安全性一直在不断发展，在云和标识管理框架中构建一种定期显示安全性发展并发现保护环境的新方法是很重要的。

标识安全分数是 Microsoft 发布的一组建议的安全控制，旨在为你提供一个数字分数，以便客观地度量你的安全状况，并帮助计划未来的安全改进。 你还可以查看你的分数与其他行业分数的比较，以及你自己的分数在一段时间内的趋势。

**最佳做法** ：根据你所在行业的最佳做法来计划例程安全评审和改进。
**详细信息** ：使用标识安全分数功能对你在一段时间内的改进进行排名。

## <a name="enable-password-management"></a>启用密码管理

如果有多个租户或者你想要允许用户[重置自己的密码](../../active-directory/user-help/active-directory-passwords-update-your-own-password.md)，则必须使用适当的安全策略来防止滥用。

**最佳做法** ：为用户设置自助式密码重置 (SSPR)。  
**详细信息** ：使用 Azure AD [自助式密码重置](/active-directory-b2c/active-directory-b2c-reference-sspr)功能。

**最佳做法** ：监视是否在使用 SSPR 及其使用情况。  
**详细信息** ：通过使用 Azure AD [密码重置注册活动报表](/active-directory/authentication/howto-sspr-reporting)监视正在注册的用户。 Azure AD 提供的报表功能可帮助使用预生成的报表来回答问题。 如果有相应的授权，还可以创建自定义查询。

## <a name="enforce-multi-factor-verification-for-users"></a>对用户强制执行多重身份验证

建议对所有用户要求进行双重验证。 这包括组织中的管理员和其他人员，如果他们的帐户泄露，可能会产生重大影响（例如，财务官员）。

要求双重验证有多种选项。 最佳选项取决于你的目标、正在运行的 Azure AD 版本以及许可计划。 请参阅[如何要求对用户进行双重验证](/active-directory/authentication/howto-mfa-userstates)了解最佳选项。 有关许可和定价的详细信息，请参阅 [Azure AD](https://www.azure.cn/pricing/details/active-directory/) 和 [Azure 多重身份验证](https://www.azure.cn/pricing/details/multi-factor-authentication/)定价页。

以下是启用双重验证的选项和优势：

**选项 1** ：使用 Azure AD 安全默认值为所有用户和登录方法启用 MFA 优势：借助此选项，可以轻松、快速地为环境中的所有用户强制执行 MFA，同时采用严格的策略来执行以下操作：

* 质询管理帐户和管理登录机制
* 要求通过 Microsoft Authenticator 对所有用户进行 MFA 质询
* 限制旧身份验证协议。

此方法可用于所有许可层，但不能与现有的条件访问策略混合使用。 你可以在 [Azure AD 安全默认值](/active-directory/fundamentals/concept-fundamentals-security-defaults)中找到更多信息

<!-- not available-->


<!-- not available-->


## <a name="use-role-based-access-control"></a>使用基于角色的访问控制

对于任何使用云的组织而言，云资源的访问管理至关重要。 [Azure 基于角色的访问控制 (Azure RBAC)](/role-based-access-control/overview) 可帮助你管理谁有权访问 Azure 资源、他们可以对这些资源执行哪些操作以及他们有权访问哪些区域。

在 Azure 中指定负责特定功能的组或单个角色有助于避免混乱，从而避免可能会导致安全风险的人为错误和自动化错误。 对于想要实施数据访问安全策略的组织而言，必须根据“需要知道”和“最低权限”安全原则限制访问权限。

你的安全团队需要能够洞察你的 Azure 资源，以便评估并修正风险。 如果安全团队具有运营职责，则需要额外的权限来完成他们的作业。

可以使用 [RBAC](/role-based-access-control/overview) 在一定范围内向用户、组和应用分配权限。 角色分配的范围可以是订阅、资源组或单个资源。

**最佳做法** ：在团队中分离职责，只向用户授予执行作业所需的访问权限。 只允许在特定范围内执行特定操作，而不要在 Azure 订阅或资源中向每个人都授予无限制权限。
**详细信息** ：使用 Azure 中的 [Azure 内置角色](/role-based-access-control/built-in-roles)向用户分配权限。

> [!Note]
> 特定的权限会造成不必要的复杂性和混乱，累积成一个“遗留”配置，难以在不担心破坏某些东西的情况下修复。 避免特定于资源的权限。 而是将管理组用于企业范围内的权限，并将资源组用于订阅中的权限。 避免用户特定的权限。 而是向 Azure AD 中的组分配权限。

**最佳做法** ：向具有 Azure 职责的安全团队授予对 Azure 资源的访问权限，以便他们可以评估和修正风险。
**详细信息** ：向安全团队授予 RBAC [安全读取者](/role-based-access-control/built-in-roles#security-reader)角色。 可以使用根管理组或段管理组，具体视职责范围而定：

* 根管理组：用于负责所有企业资源的团队
* 段管理组：用于范围有限的团队（通常是由于法规或其他组织边界所致）

**最佳做法** ：向具有直接运营职责的安全团队授予适当的权限。
**详细信息** ：审阅 RBAC 内置角色，以进行适当的角色分配。 如果内置角色不能满足组织的具体需求，则可以创建 [Azure 自定义角色](/role-based-access-control/custom-roles)。 与内置角色一样，可以在订阅、资源组和资源范围内向用户、组和服务主体分配自定义角色。

**最佳做法** ：向需要的安全角色授予 Azure 安全中心访问权限。 使用安全中心，安全团队可以快速发现和修正风险。
**详细信息** ：将具有这些需求的安全团队添加到 RBAC [安全管理员](/role-based-access-control/built-in-roles#security-admin)角色，这样他们就可以查看安全策略、查看安全状态、编辑安全策略、查看警报和建议，并能消除警报和建议。 你可以使用根管理组或段管理组来执行此操作，具体取决于职责范围。

未使用 RBAC 等功能实施数据访问控制的组织可能会给其用户分配不必要的权限。 允许用户访问他们不应有权访问的类型的数据（例如，对业务有重大影响的数据）可能会导致数据泄漏。

<!--Lower exposure not available -->
## <a name="control-locations-where-resources-are-created"></a>控制创建资源的位置

非常重要的一点是，既要允许云操作员执行任务，同时又要防止他们违反管理组织资源所需的惯例。 想要控制创建资源的位置的组织应该对这些位置进行硬编码。

可以使用 [Azure 资源管理器](/azure-resource-manager/resource-group-overview)创建安全策略，其中的定义描述了会明确遭到拒绝的操作或资源。 可以在所需范围（例如订阅、资源组或是单个资源）分配这些策略定义。

> [!NOTE]
> 安全策略与 RBAC 不同。 它们实际上使用 RBAC 授权用户来创建这些资源。
>
>

无法控制资源创建方式的组织更容易因用户创建的资源超过所需数目，而产生滥用服务的情况。 强化资源创建过程是保护多租户方案的重要步骤。

<!--not available -->
## <a name="use-azure-ad-for-storage-authentication"></a>使用 Azure AD 进行存储身份验证
[Azure 存储](/storage/common/storage-auth-aad)支持使用 Azure AD 对 Blob 存储和队列存储进行身份验证和授权。 借助 Azure AD 身份验证，可以使用基于 Azure 角色的访问控制向用户、组和应用（一直到各个 Blob 容器或队列的范围）授予特定权限。

建议使用 [Azure AD 验证对存储的访问](https://azure.microsoft.com/blog/azure-storage-support-for-azure-ad-based-access-control-now-generally-available/)。

## <a name="next-step"></a>后续步骤

有关通过 Azure 设计、部署和管理云解决方案时可以使用的更多安全最佳做法，请参阅 [Azure 安全最佳做法和模式](best-practices-and-patterns.md)。

