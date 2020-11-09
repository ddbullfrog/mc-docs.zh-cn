---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 管理你的订阅 - Azure Databricks
description: 了解如何管理你的 Azure Databricks 帐户订阅。
ms.openlocfilehash: dbdceda06bbdeb17528c0614c2189121a7f7f2df
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106485"
---
# <a name="manage-your-subscription"></a>管理订阅

Azure 帐户由 Azure 订阅信息和关联的服务组成。

## <a name="assign-account-admins"></a><a id="assign-account-admins"> </a><a id="assign-initial-account-admins"> </a>分配帐户管理员

创建 Azure Databricks 工作区的用户会被自动分配为 Azure Databricks 工作区中的管理员。 此外，当用户单击“启动工作区”时，会将 Azure 门户中在 Azure Databricks 工作区上具有“参与者”或“所有者”角色的任何用户创建为工作区中的管理员。 有关 Azure 角色的更多详细信息，请参阅[在 Azure 门户中使用 RBAC 管理访问](/role-based-access-control/role-assignments-portal)。

> [!div class="mx-imgBorder"]
> ![访问控制](../../_static/images/admin-settings/azure-portal-iam-permissions.png)

创建了工作区的用户通常在工作区中具有“参与者”或“所有者”角色。
如果没有帐户管理员可以登录到工作区（例如，当这些帐户管理员不再在你的 Active Directory 中时），则你的订阅管理员可以通过分配“参与者”或“所有者”角色向帐户管理员授予访问权限。

在帐户管理员登录到 Azure Databricks 后，他们可以在[管理控制台](../admin-console.md)中添加其他用户（管理员和非管理员）。

> [!WARNING]
>
> 不应将“参与者”或“所有者”角色分配给不应具有工作区管理员访问权限的用户。

## <a name="upgrade-or-downgrade-an-azure-databricks-workspace"></a><a id="upgrade-downgrade"> </a><a id="upgrade-or-downgrade-an-azure-databricks-workspace"> </a>升级或降级 Azure Databricks 工作区

Azure Databricks 提供了两个[定价选项](https://azure.microsoft.com/pricing/details/databricks/)：“标准”和“高级”，它们提供的功能适用于不同类型的工作负荷。 你在创建 Azure Databricks 工作区时指定一个选项。 如果你对工作区中所需的功能改变主意，可以更改其定价层。 本部分介绍了如何将 Azure Databricks 工作区从标准版升级到高级版，以及从高级版降级到标准版。 可以使用 Azure 门户、Azure 资源管理器 (ARM) 模板或 Azure REST API 和 CLI 执行升级或降级操作。

> [!NOTE]
>
> 升级或降级工作区时，会保留笔记本、用户和群集配置，但活动群集可能会被终止。

### <a name="azure-portal"></a>Azure 门户

若要在 Azure 门户中升级或降级 Azure Databricks 工作区定价选项，请使用用来新建 Databricks 工作区的[同一过程](/azure-databricks/quickstart-create-databricks-workspace-portal)。

如果已存在标准工作区，请使用用于标准工作区的相同参数将其重新创建为高级工作区：

* 名称
* 订阅
* 资源组（如果在最初部署工作区时创建了新的资源组，请在升级过程中选择该资源组作为现有资源组）
* 位置

对于“定价层”，请选择“高级”。

例如，假设你最初使用以下参数创建了一个标准工作区：

> [!div class="mx-imgBorder"]
> ![标准工作区](../../_static/images/account-settings/standard-ws.png)

若要升级到高级版，请使用以下参数重新创建工作区：

> [!div class="mx-imgBorder"]
> ![高级工作区](../../_static/images/account-settings/upgrade-to-premium-ws.png)

若要从高级版降级到标准版，请按相同的过程操作，并选择“标准”定价层。

### <a name="arm-template"></a>ARM 模板

若要升级，请使用 [基本模板](https://azure.microsoft.com/resources/templates/101-databricks-workspace/)或 [自定义 CIDR 范围](https://azure.microsoft.com/resources/templates/101-databricks-workspace-with-custom-vnet-address/)模板，具体取决于标准工作区的用途。 使用完全相同的参数重新创建工作区（如果使用自定义 CIDR 范围模板，则还使用相同的 CIDR 范围）。 将 `pricingTier` 参数设置为 Premium。

若要从高级版降级到标准版，请按相同的过程操作并将 `pricingTier` 设置为 Standard。

### <a name="rest-apicli"></a>REST API/CLI

若要进行升级，请通过 [Azure API](https://docs.microsoft.com/rest/api/databricks/workspaces/createorupdate) 使用与标准工作区完全相同的参数重新创建工作区，并将 `sku` 属性指定为 Premium。 若要使用该 API，必须将客户端应用程序注册到 Azure AD，并[获取一个访问令牌](https://docs.microsoft.com/rest/api/azure/)。

你还可以使用 Azure CLI [资源更新](https://docs.microsoft.com/cli/azure/resource?view=azure-cli-latest#az-resource-update)命令执行升级。 你不需要单独注册客户端应用程序或获取访问令牌。

若要从高级版降级到标准版，请按相同的过程操作并将 `sku` 属性指定为 Standard。

## <a name="delete-an-azure-databricks-service"></a>删除 Azure Databricks 服务

若要删除 Azure Databricks 服务，请执行以下操作：

1. 以帐户所有者（创建了服务的用户）身份登录到 Azure Databricks 工作区，然后单击右上方的用户配置文件 ![帐户图标](../../_static/images/account-settings/account-icon.png) 图标。

   > [!div class="mx-imgBorder"]
   > ![用户配置文件](../../_static/images/account-settings/manage-account.png)

2. 选择“管理帐户”。

   > [!div class="mx-imgBorder"]
   > ![管理帐户](../../_static/images/account-settings/azure-databricks-service.png)

3. 在 Azure Databricks 服务中，单击 ![Azure 删除](../../_static/images/account-settings/azure-delete.png)，然后单击“确定”。

## <a name="cancel-an-azure-subscription"></a>取消 Azure 订阅

若要取消你的 Azure 订阅，请参阅[取消 Azure 订阅](/billing/billing-how-to-cancel-azure-subscription)。