---
title: 教程 - 创建 Azure Active Directory B2C 租户
description: 按照本教程进行操作，了解如何通过使用 Azure 门户创建 Azure Active Directory B2C 租户来准备注册应用程序。
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 11/04/2020
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: d777659286adfc5fbb000f8d142b81a29ce67d7a
ms.sourcegitcommit: 33f2835ec41ca391eb9940edfcbab52888cf8a01
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94326444"
---
# <a name="tutorial-create-an-azure-active-directory-b2c-tenant"></a>教程：创建 Azure Active Directory B2C 租户

必须先在你管理的租户中注册应用程序，然后这些应用程序才能与 Azure Active Directory B2C (Azure AD B2C) 交互。

在本文中，学习如何：

> [!div class="checklist"]
> * 创建 Azure AD B2C 租户
> * 将租户链接到订阅
> * 切换到包含 Azure AD B2C 租户的目录
> * 在 Azure 门户中将 Azure AD B2C 资源添加为“收藏夹”中的项

在下一个教程中，学习如何注册应用程序。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="create-an-azure-ad-b2c-tenant"></a>创建 Azure AD B2C 租户

1. 登录 [Azure 门户](https://portal.azure.cn/)。 使用至少在订阅中分配了[参与者](../role-based-access-control/built-in-roles.md)角色的 Azure 帐户或订阅中的资源组登录。

1. 选择包含订阅的目录。

    在 Azure 门户工具栏中，选择“目录 + 订阅”图标，然后选择包含订阅的目录。 此目录不同于将包含 Azure AD B2C 租户的其他目录。

    ![订阅租户，“目录”+“订阅”筛选器，订阅租户为选中状态](./media/tutorial-create-tenant/portal-01-pick-directory.png)

1. 在 Azure 门户菜单或“主页”页上，选择“创建资源” 。
1. 搜索“Azure Active Directory B2C”，然后选择“创建” 。
1. 选择“创建新的 Azure AD B2C 租户”。

    ![创建在 Azure 门户中选择的新 Azure AD B2C 租户](./media/tutorial-create-tenant/portal-02-create-tenant.png)
    
1. 输入 **组织名称** 和 **初始域名** 。 选择 **国家或地区** （以后不能更改），然后选择“创建”。

    域名将用作完整租户域名的一部分。 在此示例中，租户名称是 *contosob2c.partner.onmschina.cn* ：

1. 租户创建过程完成后，请在租户创建页面的顶部选择“创建新的 B2C 租户或链接到现有租户”链接。

    ![Azure 门户中突出显示的“链接租户”痕迹导航链接](./media/tutorial-create-tenant/portal-04-select-link-sub-link.png)

1. 选择“将现有 Azure AD B2C 租户链接到我的 Azure 订阅”。 需要以所有者角色的身份登录才能执行此步骤。

   ![Azure 门户中“链接现有订阅”的选项](./media/tutorial-create-tenant/portal-05-link-subscription.png)

1. 选择创建的 **Azure AD B2C 租户** ，然后选择你的 **订阅** 。

    对于“资源组”，选择“新建”。 输入要包含该租户的资源组的 **名称** ，选择 **资源组位置** ，然后选择“创建”。

    ![Azure 门户中的“链接订阅”设置窗体](./media/tutorial-create-tenant/portal-06-link-subscription-settings.png)
    

可以将多个 Azure AD B2C 租户链接到单个 Azure 订阅以进行计费。 若要链接租户，你必须是 Azure AD B2C 租户中的管理员，并且在 Azure 订阅中分配有至少一个参与者角色。 请参阅[将 Azure AD B2C 租户链接到订阅](billing.md#link-an-azure-ad-b2c-tenant-to-a-subscription)。

## <a name="select-your-b2c-tenant-directory"></a>选择 B2C 租户目录

若要开始使用新的 Azure AD B2C 租户，需要切换到包含该租户的目录。

在 Azure 门户的顶部菜单中选择“目录 + 订阅”筛选器，然后选择包含 Azure AD B2C 租户的目录。

如果一开始在列表中没有看到新的 Azure B2C 租户，请刷新浏览器窗口，然后在顶部菜单中再次选择“目录 + 订阅”筛选器。

![包含在 Azure 门户中选择的目录的 B2C 租户](./media/tutorial-create-tenant/portal-07-select-tenant-directory.png)

## <a name="add-azure-ad-b2c-as-a-favorite-optional"></a>将 Azure AD B2C 添加为收藏项（可选）

此可选步骤使你更容易在以下及所有后续教程中选择 Azure AD B2C 租户。

不用在每次想使用租户时在所有服务中搜索 Azure AD B2C，而是可以将该资源纳入收藏夹。 然后，可以从门户菜单的“收藏夹”部分中选择它，快速浏览到 Azure AD B2C 租户。

只需执行此操作一次。 在执行这些步骤之前，确保已切换到包含 Azure AD B2C 租户的目录，如前一节[选择 B2C 租户目录](#select-your-b2c-tenant-directory)中所述。

1. 登录 [Azure 门户](https://portal.azure.cn)。
1. 在 Azure 门户菜单中，选择“所有服务”。
1. 在“所有服务”搜索框中，搜索“Azure AD B2C”，将鼠标悬停在搜索结果上，然后选择工具提示中的星形图标 。 Azure AD B2C 现在显示在 Azure 门户的“收藏夹”下 。
1. 如果要更改新收藏内容的位置，转到 Azure 门户菜单，选择 Azure AD B2C，然后将其上下拖动到所需位置。

    ![Azure AD B2C、“收藏夹”菜单、Azure 门户](./media/tutorial-create-tenant/portal-08-b2c-favorite.png)

## <a name="next-steps"></a>后续步骤

本文介绍了如何执行以下操作：

> [!div class="checklist"]
> * 创建 Azure AD B2C 租户
> * 将租户链接到订阅
> * 切换到包含 Azure AD B2C 租户的目录
> * 在 Azure 门户中将 Azure AD B2C 资源添加为“收藏夹”中的项

接下来，了解如何在新租户中注册 web 应用程序。

> [!div class="nextstepaction"]
> [注册应用程序 >](tutorial-register-applications.md)

