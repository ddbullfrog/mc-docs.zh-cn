---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/14/2020
title: 启用工作区对象访问控制 - Azure Databricks
description: 了解如何为 Azure Databricks 工作区对象（例如文件夹、笔记本、MLflow 试验和 MLflow 模型）启用和禁用访问控制功能。
keyword: workspace acl
ms.openlocfilehash: 97381316587c3e9d45c299a9e689234ed3156156
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106486"
---
# <a name="enable-workspace-object-access-control"></a><a id="enable-workspace-object-access-control"> </a><a id="workspace-acl"> </a>启用工作区对象访问控制

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中提供。

默认情况下，除非管理员启用工作区访问控制，否则所有用户均可创建和修改[工作区](../../workspace/index.md)对象（包括文件夹、笔记本、试验和模型）。 在使用工作区访问控制的情况下，用户的操作能力取决于单个权限。 本文介绍如何启用工作区访问控制，以及如何防止用户看到他们无权访问的工作区对象。

有关分配权限和配置工作区对象访问控制的信息，请参阅[工作区对象访问控制](../../security/access-control/workspace-acl.md)。

## <a name="enable-workspace-object-access-control"></a><a id="enable-workspace-acl"> </a><a id="enable-workspace-object-access-control"> </a>启用工作区对象访问控制

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“访问控制”选项卡](../../_static/images/admin-settings/access-control-tab-azure.png)

3. 单击“工作区访问控制”旁边的“启用”按钮。
4. 单击“确认”以确认更改。

## <a name="prevent-users-from-seeing-workspace-objects-they-do-not-have-access-to"></a><a id="prevent-users-from-seeing-workspace-objects-they-do-not-have-access-to"> </a><a id="workspace-object-visibility"> </a>防止用户看到他们无权访问的工作区对象

工作区访问控制本身不会阻止用户看到 Azure Databricks UI 中显示的工作区对象的文件名，即使用户没有这些工作区对象的权限。 若要防止笔记本文件名和文件夹显示给没有权限的用户，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。
3. 单击“工作区可见性控制”旁边的“启用”按钮 。

## <a name="library-and-jobs-access-control"></a>库和作业访问控制

![库](../../_static/images/access-control/library.png) 所有用户均可查看库。 若要控制谁可以将库附加到群集，请参阅[群集访问控制](../../security/access-control/cluster-acl.md)。

![作业](../../_static/images/access-control/jobs.png) 若要启用作业访问控制和作业可见性访问控制，请参阅[为工作区启用作业访问控制](jobs-acl.md)。 若要控制谁可以运行作业并查看作业运行结果，请参阅[作业访问控制](../../security/access-control/jobs-acl.md)。