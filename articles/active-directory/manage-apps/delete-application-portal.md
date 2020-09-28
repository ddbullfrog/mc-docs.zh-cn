---
title: 快速入门：从 Azure Active Directory (Azure AD) 租户中删除应用程序
description: 本快速入门使用 Azure 门户从 Azure Active Directory (Azure AD) 租户中删除应用程序。
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 09/23/2020
ms.author: v-junlch
ms.openlocfilehash: 0cd82d33bdf8f745fd9d86068a7ee89987b4d6e0
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245613"
---
# <a name="quickstart-delete-an-application-from-your-azure-active-directory-azure-ad-tenant"></a>快速入门：从 Azure Active Directory (Azure AD) 租户中删除应用程序

本快速入门使用 Azure 门户删除已添加到 Azure Active Directory (Azure AD) 租户的应用程序。

## <a name="prerequisites"></a>先决条件

若要从 Azure AD 租户中删除应用程序，需具备：

- 具有活动订阅的 Azure 帐户。 [创建帐户](https://www.azure.cn/pricing/1rmb-trial)。
- 以下角色之一：全局管理员、云应用程序管理员、应用程序管理员或服务主体的所有者。
- 可选：完成[查看应用](view-applications-portal.md)。
- 可选：完成[配置应用](add-application-portal-configure.md)。

>[!IMPORTANT]
>使用非生产环境测试本快速入门中的步骤。

## <a name="delete-an-application-from-your-azure-ad-tenant"></a>从 Azure AD 租户中删除应用程序

从 Azure AD 租户中删除应用程序的方法如下：

1. 在 Azure AD 门户中，选择“企业应用程序”。 然后找到并选择要删除的应用程序。 在这种情况下，我们会删除在前面的快速入门中添加的 GitHub_test 应用程序。
1. 在左侧窗格的“管理”部分中，选择“属性” 。
1. 选择“删除”，然后选择“是”以确认要从 Azure AD 租户中删除该应用 。


## <a name="clean-up-resources"></a>清理资源

完成本快速入门系列后，请考虑删除应用以清理测试租户。 本快速入门介绍了如何删除应用。


