---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/19/2020
title: 池访问控制 - Azure Databricks
description: 了解如何管理对 Azure Databricks 池的访问，这些池是可用于启动群集的一组空闲的现成可用实例。
ms.openlocfilehash: 0eeb4a1731ce6693c1a6da1fb7653512c94ebdbf
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937789"
---
# <a name="pool-access-control"></a>池访问控制

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中可用。

默认情况下，除非管理员启用池访问控制，否则所有用户均可创建和修改[池](../../clusters/instance-pools/index.md)。 使用池访问控制，用户的操作能力取决于权限。 本文介绍各个权限以及配置池访问控制的方式。

Azure Databricks 管理员必须先为工作区启用池访问控制，然后你才能使用它。 请参阅[为工作区启用池访问控制](../../administration-guide/access-control/pool-acl.md)。

## <a name="pool-permissions"></a>池权限

池权限级别分为三个：“无权限”、“可附加到”和“可管理”  。 该表列出了每个权限赋予用户的能力。

| 能力                               | 无权限    | 可附加到     | 可管理    |
|---------------------------------------|-------------------|-------------------|---------------|
| 将群集附加到池                |                   | x                 | x             |
| 删除池                           |                   |                   | x             |
| 编辑池                             |                   |                   | x             |
| 修改池权限               |                   |                   | x             |

## <a name="configure-pool-permissions"></a>配置池权限

若要向用户或组授予使用 UI 来管理池或将群集附加到池的权限，请在池配置页面的底部，选择“权限”选项卡。方法：

* 从“选择用户或组”下拉列表中选择用户和组，并为其分配权限级别。
* 使用用户或组名称旁边的下拉菜单，为已添加的用户和组更新池权限。

> [!div class="mx-imgBorder"]
> ![分配池权限](../../_static/images/access-control/pool-permissions.png)

> [!NOTE]
>
> 还可以使用[权限 API](../../_static/api-refs/permissions-azure.yaml) 授予用户或组管理池或将群集附加到池的权限。

只能通过 SCIM API 授予用户或组创建池的权限。 请遵循 [SCIM API](../../dev-tools/api/latest/scim/index.md) 文档并向用户授予 `allow-instance-pool-create` 权限。