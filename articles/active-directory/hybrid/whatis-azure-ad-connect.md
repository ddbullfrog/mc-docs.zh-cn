---
title: 什么是 Azure AD Connect。 | Microsoft Docs
description: 了解用来通过 Azure AD 同步和监视本地环境的工具。
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
ms.openlocfilehash: f4fdf35c54174fa206759633efb0c39a97e98fdd
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245526"
---
# <a name="what-is-azure-ad-connect"></a>什么是 Azure AD Connect？

Azure AD Connect 专用于满足和完成混合标识目标的 Microsoft 工具。  它提供以下功能：
     
- [密码哈希同步](whatis-phs.md) - 一种登录方法，它将用户的本地 AD 密码与 Azure AD 进行同步。
- [联合身份验证集成](how-to-connect-fed-whatis.md) - 联合身份验证是 Azure AD Connect 的可选部件，可用于使用本地 AD FS 基础结构配置混合环境。 它还提供了 AD FS 管理功能，例如证书续订和其他 AD FS 服务器部署。
- [同步](how-to-connect-sync-whatis.md) - 负责创建用户、组和其他对象。  另外，它还负责确保本地用户和组的标识信息与云匹配。  此同步还包括密码哈希。


![什么是 Azure AD Connect](./media/whatis-hybrid-identity/arch.png)

## <a name="why-use-azure-ad-connect"></a>为何使用 Azure AD Connect？
将本地目录与 Azure AD 集成可提供用于访问云和本地资源的通用标识，来提高用户的工作效率。 用户和组织可以得到以下好处：

* 用户可以使用单个标识来访问本地应用程序和云服务，例如 Microsoft 365。
* 单个工具即可提供轻松同步和登录的部署体验。
* 为方案提供最新功能。 Azure AD Connect 取代了 DirSync 和 Azure AD Sync 等早期版本的标识集成工具。 

## <a name="license-requirements-for-using-azure-ad-connect"></a>使用 Azure AD Connect 的许可证要求

[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-free-license.md)]

## <a name="next-steps"></a>后续步骤

- [硬件和先决条件](how-to-connect-install-prerequisites.md) 
- [快速设置](how-to-connect-install-express.md)
- [自定义设置](how-to-connect-install-custom.md)

