---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/14/2020
title: 为工作区启用作业访问控制 - Azure Databricks
description: 为 Azure Databricks 作业启用和禁用访问控制功能。
ms.openlocfilehash: 0a8c937f5926149f3804afe2952a5db546e1b34c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106458"
---
# <a name="enable-jobs-access-control-for-your-workspace"></a>为工作区启用作业访问控制

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中可用。

默认情况下，除非管理员启用作业访问控制，否则所有用户均可创建和修改[作业](../../jobs.md)。 使用作业访问控制，用户的操作能力取决于单个权限。 本文介绍如何启用作业访问控制，以及如何防止用户看到他们无权访问的作业。

有关分配权限和配置作业访问控制的信息，请参阅[作业访问控制](../../security/access-control/jobs-acl.md)。

## <a name="enable-jobs-access-control"></a>启用作业访问控制

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“访问控制”选项卡](../../_static/images/admin-settings/access-control-tab-azure.png)

3. 单击“群集和作业访问控制”旁边的“启用”按钮 。
4. 单击“确认”以确认更改。

## <a name="prevent-users-from-seeing-jobs-they-do-not-have-access-to"></a><a id="jobs-visibility"> </a><a id="prevent-users-from-seeing-jobs-they-do-not-have-access-to"> </a>防止用户看到他们无权访问的作业

作业访问控制本身不会阻止用户看到 Azure Databricks UI 中显示的作业，即使用户没有这些作业的权限也是如此。 若要防止用户看到这些作业，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。
3. 单击“作业可见性控制”旁边的“启用”按钮 。