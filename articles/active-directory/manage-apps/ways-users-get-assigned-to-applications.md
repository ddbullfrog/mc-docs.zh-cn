---
title: 了解如何在 Azure Active Directory 中将用户分配到应用
description: 了解如何将用户分配到使用 Azure Active Directory 进行标识管理的应用。
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: v-junlch
ms.openlocfilehash: 51dacdab70942568e9aa47e3a6000fe69fc2bf18
ms.sourcegitcommit: 4d06a5e0f48472f5eadd731e43afb1e9fbba5787
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041497"
---
# <a name="understand-how-users-are-assigned-to-apps-in-azure-active-directory"></a>了解如何在 Azure Active Directory 中将用户分配到应用
本文介绍如何将用户分配给租户中的应用程序。

## <a name="how-do-users-get-assigned-to-an-application-in-azure-ad"></a>如何将用户分配给 Azure AD 中的应用程序？
用户要访问应用程序，必须先以某种方式将其分配给该应用程序。 可使用管理员、业务委托，或有时使用用户本身的身份执行分配。 下文介绍了可以将用户分配给应用程序的方式：

*  管理员直接[将用户分配给](/active-directory/manage-apps/assign-user-or-group-access-portal)应用程序
*  管理员启用[自助服务组管理](/active-directory/users-groups-roles/groups-self-service-management)以允许用户**无需业务批准**，即可加入已对其分配应用程序的组
*  管理员启用[自助服务组管理](/active-directory/users-groups-roles/groups-self-service-management)以允许用户在经过**选定业务批准者的事先批准**的情况下，加入已对其分配应用程序的组
*  对于第一方应用程序（如 [Microsoft 365](https://products.office.com/)），管理员直接为用户分配许可证
*  对于第一方应用程序（如 [Microsoft 365](https://products.office.com/)），管理员直接为此用户所在的组分配许可证
*  [管理员同意](/active-directory/develop/active-directory-devhowto-multi-tenant-overview)所有用户均可使用某个应用程序，然后用户登录该应用程序
* 通过登录应用程序，用户自己[同意使用应用程序](/active-directory/develop/active-directory-devhowto-multi-tenant-overview)

## <a name="next-steps"></a>后续步骤
* [应用程序管理的快速入门系列](view-applications-portal.md)

