---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 管理组 - Azure Databricks
description: 了解如何在 Azure Databricks 中管理组。
ms.openlocfilehash: 664d02972ec6ff0ce88dcc634944d30a02bb3e7b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106503"
---
# <a name="manage-groups"></a>管理组

使用组可以将相同的权利分配给多个用户。 作为管理员用户，你可以使用[管理控制台](../admin-console.md)、[组 API](../../dev-tools/api/latest/groups.md)、[SCIM API](../../dev-tools/api/latest/scim/index.md) 或[支持 SCIM 的标识提供者](scim/index.md)（例如 Azure Active Directory）来管理组。 本文介绍了如何使用管理控制台来管理组。

> [!div class="mx-imgBorder"]
> ![“组”选项卡](../../_static/images/admin-settings/groups-tab-azure.png)

使用管理控制台，你可以：

* 添加组。
* 将用户添加到组以及删除用户。
* 将组添加到其他组以及删除组。
* 授予和撤销为所有组成员创建群集的权限（如果已为工作区启用了[群集访问控制](../../security/access-control/cluster-acl.md)）。
* 通过将用户添加到管理员组或删除他们来管理管理员权限。 （你还可以使用[用户管理界面](users.md)将用户分配到管理员组。）

> [!NOTE]
>
> 在工作区资源上具有“参与者”或“所有者”角色的用户会自动分配给管理员组。 有关详细信息，请参阅[分配帐户管理员](../account-settings/account.md#assign-initial-account-admins)。

## <a name="add-a-group"></a><a id="add-a-group"> </a><a id="add-group"> </a>添加组

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 单击“+ 创建组”。
3. 输入组名称，然后单击“确认”。

   组名称必须是唯一的。 你无法更改组名称。 如果要更改某个组名称，则必须删除该组，然后使用新名称重新创建它。

## <a name="add-users-and-child-groups-to-a-group"></a>向组添加用户和子组

> [!NOTE]
>
> 不能向管理员组添加子组。

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 选择要更新的组。
3. 在“成员”选项卡上，单击“+添加用户或组”。
4. 在“添加用户或组”对话框中，单击向下箭头以显示用户和组的下拉列表，然后选择要添加的用户和组。
5. 单击向下箭头以隐藏下拉列表，然后单击“确认”。

## <a name="add-entitlements-to-a-group"></a>向组添加权利

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 选择要更新的组。
3. 在“权利”选项卡上，选择要向组中的所有用户授予的权利。

   “允许创建群集”是唯一可授予的权利，但将来会添加其他权利。 向用户授予此权利后，将允许用户创建和启动新群集。 你可以使用[群集级权限](../../security/access-control/cluster-acl.md)限制对现有群集的访问。

4. 在确认对话框中，单击“确认”。

## <a name="view-parent-groups"></a>查看父组

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 选择要更新的组。
3. 在“父级”选项卡上，查看你的组的父组。

## <a name="remove-a-user-or-child-group"></a><a id="remove-a-user-or-child-group"> </a><a id="remove-user-group"> </a>删除用户或子组

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 选择要更新的组。
3. 在“成员”选项卡上，找到要删除的用户或组，然后单击“操作”列中的 **X** 。
4. 单击“删除成员”以进行确认。

用户或子组将丢失此组中的成员身份所授予的所有权利和子组成员身份。 但请注意，他们可能会通过其他组或用户级别的授予中的成员身份保留这些权利。

## <a name="remove-an-entitlement"></a>删除权利

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 选择要更新的组。
3. 在“权利”选项卡上，清除要为组中的所有用户撤销的权利的复选框。
4. 在确认对话框中，单击“删除”。

组成员将失去权利，除非他们具有作为单个用户或通过其他组成员身份获得的权限。

## <a name="remove-a-group-from-its-parent-group"></a>将组从其父组中删除

1. 转到[管理控制台](../admin-console.md)，然后单击“组”选项卡。
2. 选择要更新的组。
3. 在“父级”选项卡上，找到你要从中脱离的父组，然后单击“操作”列中的 **X** 。
4. 在确认对话框中，单击“删除父级”。

分配给父组的所有权利将从组的成员中删除。 但请注意，他们可能会通过其他组或用户级别的授予中的成员身份保留这些权利。