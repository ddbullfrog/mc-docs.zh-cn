---
title: 什么是与 Azure AD 的密码哈希同步？ | Microsoft Docs
description: 介绍了密码哈希同步。
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 09/24/2020
ms.subservice: hybrid
ms.author: v-junlch
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9bef6ec6bd73a94456bd8a96534994b1d6bc73d8
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245507"
---
# <a name="what-is-password-hash-synchronization-with-azure-ad"></a>什么是与 Azure AD 的密码哈希同步？
密码哈希同步是用来实现混合标识的登录方法之一。 Azure AD Connect 将用户密码的哈希从本地 Active Directory 实例同步到基于云的 Azure AD 实例。

密码哈希同步是由 Azure AD Connect 同步实现的目录同步功能的扩展。可以使用此功能登录到 Azure AD 服务，例如 Microsoft 365。 登录到该服务时使用的密码与登录到本地 Active Directory 实例时使用的密码相同。

![什么是 Azure AD Connect](./media/how-to-connect-password-hash-synchronization/arch1.png)

密码哈希同步通过将用户需要维护的密码数目减少为一个来提供帮助。 密码哈希同步可以：

* 提升用户的生产力。
* 减少技术支持成本。  

若要在环境中使用密码哈希同步，需要：

* 安装 Azure AD Connect。  
* 配置本地 Azure Active Directory 实例与 Azure Active Directory 实例之间的目录同步。
* 启用密码哈希同步。



有关详细信息，请参阅[什么是混合标识？](whatis-hybrid-identity.md)




## <a name="next-steps"></a>后续步骤

- [什么是混合标识？](whatis-hybrid-identity.md)
- [什么是联合身份验证？](whatis-fed.md)
- [密码哈希同步的工作原理](how-to-connect-password-hash-synchronization.md)

