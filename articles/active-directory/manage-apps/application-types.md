---
title: 查看使用 Azure Active Directory 租户进行标识管理的应用
description: 了解如何查看使用 Azure Active Directory 租户进行标识管理的所有应用程序。
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: v-junlch
ms.openlocfilehash: 5e13d15efd98a4ef93361e8517fc2549bdaa9e72
ms.sourcegitcommit: 4d06a5e0f48472f5eadd731e43afb1e9fbba5787
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041433"
---
# <a name="viewing-apps-using-your-azure-ad-tenant-for-identity-management"></a>查看使用 Azure AD 租户进行标识管理的应用
[应用程序管理快速入门系列](view-applications-portal.md)介绍了基础知识。 其中介绍了如何查看使用 Azure AD 租户进行标识管理的所有应用。 本文将更深入地介绍你会找到的应用类型。

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>为什么在所有应用程序列表中出现某个特定应用程序？
筛选为“所有应用程序”时，“所有应用程序列表” 会显示租户中的每个服务主体对象。 服务主体对象以多种方式出现在此列表中：
- 注册或登录与 Azure Active Directory 集成的第三方应用程序时。 [Smartsheet](https://app.smartsheet.com/b/home) 或 [DocuSign](https://www.docusign.net/member/MemberLogin.aspx) 就是一个示例。
- Microsoft 应用，如 Microsoft 365。
- 当通过使用[应用程序注册表](/active-directory/active-directory-app-registration)创建以自定义方式开发的应用程序，来添加新应用程序注册之时
- 当通过使用 [V2.0 门户](/active-directory/develop/quickstart-register-app)创建以自定义方式开发的应用程序，来添加新应用程序注册之时
- 添加使用 Visual Studio 的 [ASP.NET 身份验证方法](https://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions)或[连接的服务](https://devblogs.microsoft.com/visualstudio/connecting-to-cloud-services/)开发的应用程序时
- 使用 [Azure AD PowerShell 模块](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0)创建服务主体对象时
- 以管理员身份[同意某应用程序](/active-directory/develop/active-directory-devhowto-multi-tenant-overview)使用租户中的数据时
- [用户同意某应用程序](/active-directory/develop/active-directory-devhowto-multi-tenant-overview)使用租户中的数据时
- 启用某些在租户中存储数据的服务时。 相应的一个示例是密码重置，密码重置是作为服务主体进行建模的，以便安全存储密码重置策略。

若要详细了解如何以及为何将应用添加到目录，请参阅[如何将应用程序添加到 Azure AD](/active-directory/develop/active-directory-how-applications-are-added)。


