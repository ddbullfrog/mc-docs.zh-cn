---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/14/2020
title: 为工作区启用群集访问控制 - Azure Databricks
description: 了解如何为 Azure Databricks 群集启用和禁用访问控制功能。
ms.openlocfilehash: d28d9e7f69c1cd8563533804c9bc0427ecc7b517
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106459"
---
# <a name="enable-cluster-access-control-for-your-workspace"></a>为工作区启用群集访问控制

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中提供。

默认情况下，除非管理员启用群集访问控制，否则所有用户均可创建和修改[群集](../../clusters/index.md)。 使用群集访问控制，用户的操作能力取决于权限。 本文介绍如何启用群集访问控制、配置群集创建权限，以及防止用户看到他们无权访问的群集。

有关分配权限和配置群集访问控制的信息，请参阅[群集访问控制](../../security/access-control/cluster-acl.md)。

## <a name="enable-cluster-access-control"></a><a id="cluster-acl-enable"> </a><a id="enable-cluster-access-control"> </a>启用群集访问控制

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“访问控制”选项卡](../../_static/images/admin-settings/access-control-tab-azure.png)

3. 单击“群集和作业访问控制”旁边的“启用”按钮 。
4. 单击“确认”以确认更改。

## <a name="prevent-users-from-seeing-clusters-they-do-not-have-access-to"></a><a id="cluster-visibility"> </a><a id="prevent-users-from-seeing-clusters-they-do-not-have-access-to"> </a>防止用户看到他们无权访问的群集

群集访问控制本身不会阻止用户看到 Azure Databricks UI 中显示的群集，即使用户没有这些群集的权限。 若要防止用户看到这些群集，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。
3. 单击“群集可见性控制”旁边的“启用”按钮 。
4. 单击“确认”以确认更改。

## <a name="configure-cluster-creation-permission"></a><a id="cluster-create-permission"> </a><a id="configure-cluster-creation-permission"> </a>配置群集创建权限

可以为单个用户或组分配“允许创建群集”权限。

若要为单个用户分配该权限，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 转到“用户”[](../users-groups/users.md)选项卡。
3. 选中用户所在行的“允许创建群集”复选框。

   > [!div class="mx-imgBorder"]
   > ![用户所在行](../../_static/images/admin-settings/users-list.png)

4. 单击“确认”以确认更改。

若要为[组](../users-groups/groups.md)分配该权限，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 转到“组”选项卡。
3. 选择要更新的组。
4. 在“权利”选项卡上，选择“允许创建群集”。

## <a name="example-using-cluster-level-permissions-to-enforce-cluster-configurations"></a><a id="cluster-config-enforce"> </a><a id="example-using-cluster-level-permissions-to-enforce-cluster-configurations"> </a>示例：使用群集级别权限强制实施群集配置

群集访问控制的一个优点是可以强制实施群集配置，使用户无法更改它们。

例如，管理员可能希望强制实施的配置包括：

* 用于成本退款的标记
* 向 Azure Data Lake Storage 进行 Azure AD 凭据直通身份验证，以控制对数据的访问
* 标准库

对于需要锁定群集配置的组织，Azure Databricks 建议使用以下工作流：

1. 对所有用户禁用“允许创建群集”。

   > [!div class="mx-imgBorder"]
   > ![“群集创建”复选框](../../_static/images/clusters/acl-allow-user.png)

2. 创建你想要用户使用的所有群集配置后，请向需要访问给定群集的用户授予“可重启”权限。 这样一来，用户无需手动设置所有配置即可随意启动和停止群集。

   > [!div class="mx-imgBorder"]
   > ![可重启](../../_static/images/clusters/acl-permission-details.png)