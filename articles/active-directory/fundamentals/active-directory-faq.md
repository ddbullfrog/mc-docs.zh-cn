---
title: 常见问题解答 (FAQ) - Azure Active Directory | Microsoft Docs
description: 有关 Azure 和 Azure Active Directory、密码管理以及应用程序访问的常见问题和解答。
services: active-directory
author: ajburnle
manager: daveba
ms.assetid: b8207760-9714-4871-93d5-f9893de31c8f
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: troubleshooting
ms.date: 09/23/2020
ms.author: v-junlch
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2eccaa20d91217f83eec3fc35584dfc1a2c8fa75
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91244639"
---
# <a name="frequently-asked-questions-about-azure-active-directory"></a>有关 Azure Active Directory 的常见问题
Azure Active Directory (Azure AD) 是综合性的标识即服务 (IDaaS) 解决方案，涉及到标识、访问管理和安全的方方面面。

有关详细信息，请参阅[什么是 Azure Active Directory？](active-directory-whatis.md)。


## <a name="access-azure-and-azure-active-directory"></a>访问 Azure 和 Azure Active Directory
**问：尝试在 Azure 门户中访问 Azure AD 时，为何出现“找不到订阅”错误？**

**答:** 若要访问 Azure 门户，每个用户都需要 Azure 订阅的权限。 如果没有付费的 Microsoft 365 或 Azure AD 订阅，则需要激活 [Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)或付费的订阅。

有关详细信息，请参阅：

* [Azure 订阅与 Azure Active Directory 的关联方式](active-directory-how-subscriptions-associated-directory.md)

---
**问：Azure AD、Microsoft 365 与 Azure 之间是什么关系？**

**答:** Azure AD 为所有 Web 服务提供通用的标识和访问功能。 不管使用的是 Microsoft 365、Azure、Intune 还是其他服务，都是在使用 Azure AD 为所有这些服务启用登录和访问管理。

可以将所有已设置为使用 Web 服务的用户定义为一个或多个 Azure AD 实例中的用户帐户。 可以在设置这些帐户时启用免费的 Azure AD 功能，例如云应用程序访问。

Azure AD 付费型服务（例如企业移动性 + 安全性）可通过综合性的企业级管理和安全解决方案来弥补其他 Web 服务（例如 Microsoft 365 和 Azure）的不足。

---

**问：所有者与全局管理员之间的差异是什么？**

**答:** 默认情况下，系统会将注册 Azure 订阅的人员指派为 Azure 资源的所有者角色。 所有者可以使用 Microsoft 帐户，也可以使用 Azure 订阅与之关联的目录中的工作或学校帐户。  此角色有权管理 Azure 门户中的服务。

如果其他人需要使用同一个订阅登录和访问服务，则可向他们分配相应的[内置角色](../../role-based-access-control/built-in-roles.md)。 有关其他信息，请参阅[使用 Azure 门户添加或删除 Azure 角色分配](../../role-based-access-control/role-assignments-portal.md)。

默认情况下，系统会将注册 Azure 订阅的人员指派为目录的全局管理员角色。 全局管理员有权访问所有 Azure AD 目录功能。 Azure AD 提供一组不同的管理员角色，用于管理目录和标识相关的功能。 这些管理员将有权访问 Azure 门户中的各种功能。 管理员的角色决定了其所能执行的操作，例如创建或编辑用户、向其他用户分配管理角色、重置用户密码、管理用户许可证，或者管理域。  有关 Azure AD 目录管理员及其角色的其他信息，请参阅[在 Azure Active Directory 中向用户分配管理员角色](active-directory-users-assign-role-azure-portal.md)和[在 Azure Active Directory 中分配管理员角色](../users-groups-roles/directory-assign-admin-roles.md)。

另外，Azure AD 付费型服务（例如企业移动性 + 安全性）可通过综合性企业级管理和安全解决方案来弥补其他 Web 服务（例如 Microsoft 365 和 Azure）的不足。

---
**问：是否可以通过报表来查看我的 Azure AD 用户许可证何时会过期？**

**答:** 不是。  此功能目前不可用。

---

## <a name="get-started-with-hybrid-azure-ad"></a>混合 Azure AD 入门


**问：如果我已被添加为协作者，该如何离开原来的租户？**

**答:** 如果被作为协作者添加到另一组织的租户，可使用右上角的“租户切换器”在租户之间切换。  目前还无法主动离开邀请组织，Microsoft 正致力于提供该功能。  在该功能可用之前，可以要求邀请阻止你将从其租户中删除。

---
**问：如何将我的本地目录连接到 Azure AD？**

**答:** 可以使用 Azure AD Connect 将本地目录连接到 Azure AD。

有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](../hybrid/whatis-hybrid-identity.md)。


**问：Azure AD 是否为组织中的用户提供自助服务门户？**

**答:** 是的，Azure AD 提供 [Azure AD 访问面板](https://account.activedirectory.windowsazure.cn/r#/applications)，方便用户使用自助服务以及进行应用程序访问。 如果你是 Microsoft 365 客户，可以在 Office 365 门户中找到许多相同的功能。

## <a name="password-management"></a>密码管理
**问：是否可以使用 Azure AD 密码写回但不使用密码同步？（在这种情况下，是否可以结合密码写回使用 Azure AD 自助服务密码重置 (SSPR)，而不将密码存储在云中？）**

**答:** 无需将 Active Directory 密码同步到 Azure AD 即可启用写回。 在联合环境中，Azure AD 单一登录 (SSO) 依赖本地目录对用户进行身份验证。 在这种情况下，并不需要在 Azure AD 中跟踪本地密码。

---
**问：需要多长时间才能将密码写回到 Active Directory 本地？**

**答:** 密码写回实时进行。

有关详细信息，请参阅[密码管理入门](../authentication/tutorial-enable-sspr.md)。

---
**问：是否可以对管理员管理的密码使用密码写回？**

**答:** 可以。如果已启用密码写回，管理员执行的密码操作将写回到用户的本地环境。  

<a name="for-more-answers-to-password-related-questions-see-password-management-frequently-asked-questions"></a>有关密码相关问题的详细解答，请参阅[密码管理常见问题](../authentication/active-directory-passwords-faq.md)。
---
**问：如果我在尝试更改 Microsoft 365/Azure AD 密码时忘记了现有的密码，该怎么办？**

**答:** 对于这种情况，有几个选项。  在可行的情况下，使用自助服务密码重置 (SSPR)。  SSPR 是否适用取决于其配置方式。  有关详细信息，请参阅[密码重置门户的工作原理](../authentication/howto-sspr-deployment.md)。

对于 Microsoft 365 用户，管理员可以使用[重置用户密码](https://support.office.com/article/Admins-Reset-user-passwords-7A5D073B-7FAE-4AA5-8F96-9ECD041ABA9C?ui=en-US&rs=en-US&ad=US)中概述的步骤重置密码。

对于 Azure AD 帐户，管理员可以使用以下选项之一重置密码：

- [在 Azure 门户中重置帐户](active-directory-users-reset-password-azure-portal.md)
- [使用 PowerShell](https://docs.microsoft.com/powershell/module/msonline/set-msoluserpassword?view=azureadps-1.0)


---
## <a name="security"></a>安全性
**问：帐户在经过特定次数的失败尝试后被锁定还是使用了更复杂的策略？**

我们使用更复杂的策略来锁定帐户。  这基于请求的 IP 和输入的密码。 锁定的持续时间也会根据存在攻击的可能性而延长。  

**问：某些（通用）密码会被拒绝并且显示消息“此密码已使用了许多次”，这是否是指当前 Active Directory 中使用的密码？**

这指的是全局通用的密码，例如“Password”和“123456”的任何变体。

**问：B2C 租户中就会阻止来自可疑来源（僵尸网络、Tor 终结点）的登录请求还是需要使用基本或高级版租户才能阻止？**

我们有一个网关，它会筛选请求并针对僵尸网络提供一定的防护，它适用于所有 B2C 租户。

## <a name="application-access"></a>应用程序访问



- - -
**问：用户如何使用 Azure AD 来登录应用程序？**

**答:** Azure AD 提供多种方式供用户查看和访问其应用程序，例如：

* Azure AD 访问面板
* Microsoft 365 应用程序启动器
* 直接登录联合应用
* 联合、基于密码或现有应用的深层链接

---
**问：如何要求访问特定应用程序的用户进行多重身份验证？**

**答:** 使用 Azure AD 条件访问，可以针对每个应用程序分配独特的访问策略。 可以在策略中要求用户始终进行多重身份验证，或者在未连接到本地网络时才进行。  

有关详细信息，请参阅[保护对 Microsoft 365 和其他连接到 Azure Active Directory 的应用的访问](../conditional-access/overview.md)。

**问：是否可以通过 Azure AD 设置安全的 LDAP 连接？**

**答:** 不是。 Azure AD 不直接支持轻型目录访问协议 (LDAP) 或安全 LDAP。 但是，可以借助 Azure 网络通过正确配置的网络安全组在 Azure AD 租户上启用 Azure AD 域服务 (Azure AD DS) 实例，以实现 LDAP 连接。 有关详细信息，请参阅[为 Azure Active Directory 域服务托管域配置安全 LDAP](../../active-directory-domain-services/tutorial-configure-ldaps.md)

