---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/09/2020
title: 为工作区启用表访问控制 - Azure Databricks
description: 了解管理员如何使用 Python 和 SQL 为 Azure Databricks 工作区启用和强制实施表访问权限。
ms.openlocfilehash: ef6cc5da8b88075d5c00ffee322628b29a4f8f38
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106528"
---
# <a name="enable-table-access-control-for-your-workspace"></a>为工作区启用表访问控制

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../release-notes/release-types.md)提供。

利用表访问控制，你可以使用基于 Azure Databricks 视图的访问控制模型以编程方式授予和撤销对数据的访问权限。 表访问控制需要 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)。

本文介绍如何为 Azure Databricks 工作区启用和强制实施 [Python 和 SQL 表访问控制](../../security/access-control/table-acls/index.md)。
若要了解如何在群集上启用表访问控制，请参阅[为群集启用表访问控制](../../security/access-control/table-acls/table-acl.md)。 若要了解启用了表访问控制后如何对数据对象设置特权，请参阅[数据对象特权](../../security/access-control/table-acls/object-privileges.md)。

若要保护从群集进行的表访问操作，另一种方法是使用[仅限 SQL 的表访问控制](../../security/access-control/table-acls/table-acl.md#sql-only-table-acl)，该方法通常可用，不需要使用本文介绍的选项来启用。

## <a name="enable-table-access-control-for-your-workspace"></a><a id="enable-table-access-control-for-your-workspace"> </a><a id="enable-table-acl"> </a>为工作区启用表访问控制

1. 登录到[管理控制台](../admin-console.md)。
2. 转到“访问控制”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“访问控制”选项卡](../../_static/images/admin-settings/access-control-tab-azure.png)

3. 确保[群集访问控制](../../security/access-control/cluster-acl.md)已启用。 必须先启用群集访问控制，然后才能启用表访问控制。
4. 在“表访问控制”旁边，单击“启用”按钮。
5. 单击“确认”  。

## <a name="enforce-table-access-control"></a>强制实施表访问控制

为了确保用户只访问你希望他们访问的数据，你必须只允许用户访问已启用表访问控制的群集。 具体而言，你应确保：

* 用户无权创建群集。 如果在没有表访问控制的情况下创建群集，则可以从该群集访问任何数据。

  > [!div class="mx-imgBorder"]
  > ![禁用群集创建权限](../../_static/images/access-control/table-acl-no-allow-cluster-create-azure.png)

* 对于任何未启用表访问控制的群集，用户没有“可附加到”权限。

有关详细信息，请参阅[群集访问控制](../../security/access-control/cluster-acl.md)。