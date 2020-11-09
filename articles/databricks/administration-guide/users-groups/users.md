---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 管理用户 - Azure Databricks
description: 了解如何在 Azure Databricks 中管理用户。
ms.openlocfilehash: 6cdc141fa7c24abba6fb3c4e75fcbe1d2515ce42
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106774"
---
# <a name="manage-users"></a>管理用户

作为管理员用户，你可以使用[管理控制台](../admin-console.md)、[SCIM API](../../dev-tools/api/latest/scim/index.md) 或[支持 SCIM 的标识提供者](scim/index.md)（例如 Azure Active Directory）来管理用户帐户。 本文介绍如何使用管理控制台来管理用户。

> [!div class="mx-imgBorder"]
> ![用户列表](../../_static/images/admin-settings/users-list.png)

可以使用管理控制台上的“用户”选项卡执行以下操作：

* 添加和删除用户。
* 授予和撤销创建群集的权限（如果已为工作区启用了[群集访问控制](../../security/access-control/cluster-acl.md)）。
* 通过选中“管理员”复选框来授予和撤销管理员权限。

在前面的示例中，William 是管理员，Greg 可以创建群集。

还可以在管理控制台的其他部分执行以下用户管理任务，详见其他文章：

* 将用户添加到组。 请参阅[管理组](groups.md)。

> [!NOTE]
>
> 在工作区资源上具有“参与者”或“所有者”角色的用户可以通过 Azure 门户以管理员身份登录。 有关详细信息，请参阅[分配帐户管理员](../account-settings/account.md#assign-initial-account-admins)。

## <a name="add-a-user"></a><a id="add-a-user"> </a><a id="add-user"> </a>添加用户

1. 转到[管理控制台](../admin-console.md)。
2. 在“用户”选项卡上，单击“添加用户”。
3. 提供用户电子邮件 ID。

   可以添加属于 Azure Databricks 工作区的 Azure Active Directory 租户的任何用户。

   > [!div class="mx-imgBorder"]
   > ![添加用户](../../_static/images/admin-settings/users-email-azure.png)

4. 如果已启用[群集访问控制](../../security/access-control/cluster-acl.md)，则会在没有群集创建权限的情况下添加用户。

## <a name="remove-a-user"></a><a id="remove-a-user"> </a><a id="remove-user"> </a>删除用户

1. 转到[管理控制台](../admin-console.md)。
2. 在“用户”选项卡上找到用户，然后单击用户行最右侧的 ![“删除用户”图标](../../_static/images/admin-settings/remove-user.png)。
3. 单击“删除用户”进行确认。