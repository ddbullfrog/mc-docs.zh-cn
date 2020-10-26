---
title: 帮助进行标识管理的 Azure 安全功能 | Azure Docs
description: 了解有助于标识管理的核心 Azure 安全功能。 了解有关单一登录和反向代理等主题的信息。
services: security
documentationcenter: na
author: Johnnytechn
manager: barbkess
editor: TomSh
ms.assetid: 5aa0a7ac-8f18-4ede-92a1-ae0dfe585e28
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/12/2020
ms.author: v-johya
origin.date: 09/19/2018
Customer intent: As an IT Pro or decision maker I am trying to learn about identity management capabilities in Azure
ms.openlocfilehash: e2423b9de530ec4c664e3025c0d683199cd02223
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128141"
---
# <a name="azure-identity-management-security-overview"></a>Azure 标识管理安全概述

 标识管理是对[安全主体](https://docs.microsoft.com/windows/security/identity-protection/access-control/security-principals)进行身份验证和授权的过程。 它还包括控制有关这些主体（标识）的信息。 安全主体（标识）可能包括服务、应用程序、用户和组等等。Microsoft 标识和访问管理解决方案可帮助 IT 部门保护对企业数据中心和云中的应用程序和资源的访问。 此类保护支持附加的验证级别，比如多重身份验证和条件访问策略。 通过高级安全报告、审核和警报来监视可疑活动，以便减少潜在的安全问题。 [Azure Active Directory Premium](/active-directory/active-directory-editions) 向数千个云软件即服务 (SaaS) 应用提供单一登录 (SSO)，并且可以用来访问在本地运行的 Web 应用。
 
利用 Azure Active Directory (Azure AD) 的安全优势，可以实现以下目的：

* 为混合企业中的每个用户创建和管理单一标识，从而保持用户、组和设备同步。 
* 提供对应用程序（包括数千个预先集成的 SaaS 应用）的 SSO 访问。
* 通过对本地应用程序和云应用程序实施基于规则的多重身份验证，启用应用程序访问安全措施。
* 通过 Azure AD 应用程序代理预配对本地 Web 应用程序的安全远程访问。

本文旨在概述可帮助进行标识管理的核心 Azure 安全功能。 此外还提供了文章链接，更详细说明每项功能。  

本文重点介绍以下核心 Azure 标识管理功能︰

<!--Not available in MC: Single sign-on, Reverse proxy, Device registration -->

* 多重身份验证
* Azure 基于角色的访问控制 (Azure RBAC)
* 安全监控、警报和基于机器学习的报告
* 消费者标识和访问管理
* Privileged identity management
* 标识保护
* 混合标识管理/Azure AD Connect
* Azure AD 访问评审

<!-- SSO not available -->
<!--Not available in MC: Reverse proxy-->
## <a name="multi-factor-authentication"></a>多重身份验证

Azure 多重身份验证是需要使用多个验证方法的身份验证方法，为用户登录和事务额外提供一层重要的安全保障。 多重身份验证可帮助保护对数据和应用程序的访问，同时可以满足用户对简单登录过程的需求。 它通过各种验证选项（例如电话、短信、移动应用通知或验证码以及第三方 OAuth 令牌）来提供强身份验证。

了解详细信息：

* [多重身份验证](https://docs.azure.cn/zh-cn/active-directory/authentication/)
* [什么是 Azure 多重身份验证？](/active-directory/authentication/multi-factor-authentication)
* [Azure 多重身份验证的工作原理](../../active-directory/authentication/concept-mfa-howitworks.md)

## <a name="azure-rbac"></a>Azure RBAC

Azure RBAC 是在 Azure 资源管理器基础上构建的授权系统，针对 Azure 中的资源提供精细的访问权限管理。 可以通过 Azure RBAC 精确控制用户具有的访问权限级别。 例如，可以限制一位用户仅管理虚拟网络，限制另一位用户管理资源组中的所有资源。 Azure 包含多个可用的内置角色。 下面列出了四个基本的内置角色。 前三个角色适用于所有资源类型。

- [所有者](/role-based-access-control/built-in-roles#owner) - 拥有对所有资源的完全访问权限，包括将访问权限委派给其他用户的权限。 
- [参与者](/role-based-access-control/built-in-roles#contributor) - 可以创建和管理所有类型的 Azure 资源，但无法将访问权限授予其他用户。
- [读取者](/role-based-access-control/built-in-roles#reader) - 可以查看现有的 Azure 资源。
- [用户访问管理员](/role-based-access-control/built-in-roles#user-access-administrator) - 可以管理用户对 Azure 资源的访问。

了解详细信息：

* [什么是 Azure 基于角色的访问控制 (Azure RBAC)？](/role-based-access-control/overview)
* [Azure 内置角色](/role-based-access-control/built-in-roles)

## <a name="security-monitoring-alerts-and-machine-learning-based-reports"></a>安全监控、警报和基于机器学习的报告

安全监控、警报和基于机器学习的报告（用于标识不一致的访问模式）可以帮助保护业务。 可以使用 Azure AD 的访问和使用情况报告来监控组织目录的完整性和安全性。 使用此信息，目录管理员可以更好地确定哪里可能存在安全风险，以便制定相应的计划来降低这些风险。

在 Azure 门户中，报告分为以下类别：

* **异常报告**：包含我们发现存在异常的登录事件。 我们的目标是让你知道这类活动，并让你能够确定事件是否可疑。
* **集成式应用程序报告**：就组织如何使用云应用程序提供见解。 Azure AD 提供与数千个云应用程序的集成。
* **错误报告**：指示在为外部应用程序预配帐户时可能发生的错误。
* **用户特定的报告**：显示特定用户的设备登录活动数据。
* **活动日志**：包含过去 24 小时、过去 7 天或过去 30 天内的所有已审核事件的记录，以及组活动更改记录、密码重置和注册活动记录。

了解详细信息：

* [查看访问和使用情况报告](/active-directory/active-directory-view-access-usage-reports)
* [Azure Active Directory 报告入门](/active-directory/active-directory-reporting-getting-started)
* [Azure Active Directory 报告指南](/active-directory/active-directory-reporting-guide)

## <a name="consumer-identity-and-access-management"></a>消费者标识和访问管理

Azure AD B2C 是一项高度可用的全局性标识管理服务，适用于面向用户且可通过缩放来处理数亿标识的应用程序。 它可以跨移动平台和 Web 平台进行集成。 使用者只需使用现有社交帐户或创建新凭据，即可通过可自定义的体验登录到所有应用程序。

过去，想要在自己的应用程序中注册客户并使其登录的应用程序开发人员会编写自己的代码。 他们使用本地数据库或系统存储用户名和密码。 Azure AD B2C 通过基于标准的安全平台和大量的可扩展策略，向组织提供一种更好的方式将用户标识管理集成到应用程序中。

当你使用 Azure AD B2C 时，你的用户可通过使用其现有的社交帐户或通过创建新的凭据（电子邮件地址和密码，或者用户名和密码）来注册应用程序。

了解详细信息：

* [什么是 Azure Active Directory B2C？](https://www.azure.cn/home/features/active-directory-b2c/)
* [Azure Active Directory B2C 预览版：在应用程序中注册用户和让用户登录](../../active-directory-b2c/overview.md)
* [Azure Active Directory B2C 预览版：应用程序的类型](../../active-directory-b2c/application-types.md)

<!-- Device registration not available-->
## <a name="privileged-identity-management"></a>Privileged identity management

利用 Azure AD Privileged Identity Management，你可以管理、控制和监视特权标识以及对 Azure AD 和其他 Microsoft Online Services（如 Microsoft 365 和 Microsoft Intune）中的资源的访问。

用户有时候需要在 Azure 或 Microsoft 365 资源或者其他 SaaS 应用中执行特权操作。 这种需要通常意味着，组织必须授予用户永久的 Azure AD 访问特权。 此类访问会给云中托管的资源不断增大安全风险，因为组织无法充分监视这些用户正在使用管理员特权执行哪些操作。 此外，如果有访问特权的用户帐户被泄露，此安全漏洞可能会影响组织的总体云安全性。 Azure AD Privileged Identity Management 可帮助解决这一风险。

使用 Azure AD Privileged Identity Management 可执行以下操作：

* 查看哪些用户是 Azure AD 管理员。
* 按需启用对 Microsoft 365 和 Intune 等 Microsoft 服务的实时 (JIT) 管理访问权限。
* 获取有关管理员访问历史记录以及管理员分配更改的报告。
* 获取有关访问特权角色的警报。

了解详细信息：

* [什么是 Azure AD Privileged Identity Management？](../../active-directory/privileged-identity-management/pim-configure.md)
* [在 PIM 中分配 Azure AD 目录角色](../../active-directory/privileged-identity-management/pim-how-to-add-role-to-user.md)

## <a name="identity-protection"></a>标识保护

Azure AD 标识保护是一种安全服务，它提供一个综合视图，你可以在其中查看影响组织标识的风险检测和潜在漏洞。 “标识保护”使用现有的 Azure AD 异常检测功能，该功能可通过 Azure AD 异常活动报告得到。 “标识保护”还引入了新的可以实时检测异常的风险检测类型。

<!-- not available -->
## <a name="hybrid-identity-managementazure-ad-connect"></a>混合标识管理/Azure AD Connect

Microsoft 的标识解决方案跨越本地和基于云的功能，创建单一用户标识对所有资源进行身份验证和授权，而不考虑其位置。 我们称此为混合标识。 Azure AD Connect 专用于满足和完成混合标识目标的 Microsoft 工具。 这样，便可为集成到 Azure AD 的 Microsoft 365、Azure 和 SaaS 应用程序的用户提供一个通用标识。 它提供以下功能：

* 同步
* AD FS 和联合集成
* 直通身份验证
* 运行状况监视

了解详细信息：

* [混合标识白皮书](https://download.microsoft.com/download/D/B/A/DBA9E313-B833-48EE-998A-240AA799A8AB/Hybrid_Identity_White_Paper.pdf)
* [Azure Active Directory](/active-directory/)
* [Azure AD 团队博客](https://blogs.technet.microsoft.com/ad/)

## <a name="azure-ad-access-reviews"></a>Azure AD 访问评审

Azure Active Directory (Azure AD) 访问评审可以使组织有效地管理组成员身份、对企业应用程序的访问权限，以及特权角色分配。

了解详细信息：

* [Azure AD 访问评审](../../active-directory/governance/access-reviews-overview.md)
* [使用 Azure AD 访问评审管理用户访问权限](../../active-directory/governance/access-reviews-overview.md)

