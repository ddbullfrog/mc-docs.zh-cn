---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/19/2020
title: 作业访问控制 - Azure Databricks
description: 了解如何管理对 Azure Databricks 作业的访问。
ms.openlocfilehash: c51fad3a0f2ce9f02ba6920a65fc9d4642e3803a
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937790"
---
# <a name="jobs-access-control"></a><a id="job-acl"> </a><a id="jobs-access-control"> </a>作业访问控制

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中可用。

默认情况下，除非管理员启用作业访问控制，否则所有用户均可创建和修改[作业](../../jobs.md)。 使用作业访问控制，用户的操作能力取决于单个权限。 本文介绍各个权限以及启用和配置作业访问控制的方式。

Azure Databricks 管理员必须先为工作区启用作业访问控制，然后你才能使用表访问控制。 请参阅[为工作区启用作业访问控制](../../administration-guide/access-control/jobs-acl.md)。

## <a name="job-permissions"></a>作业权限

有五个作业权限级别：无权限、可以查看、可以管理运行、是所有者、可以管理    。 默认情况下，会向管理员授予“可以管理”权限，并且他们可将该权限分配给非管理员用户。

该表列出了每个权限赋予用户的能力。

| 能力                                     | 无权限   | 可以查看   | 可以管理运行   | 为所有者   | 可管理           |
|---------------------------------------------|------------------|------------|------------------|------------|----------------------|
| 查看作业详细信息及设置               | x                | x          | x                | x          | x                    |
| 查看结果、Spark UI、作业运行日志   |                  | x          | x                | x          | x                    |
| 立即运行                                     |                  |            | x                | x          | x                    |
| 取消运行                                  |                  |            | x                | x          | x                    |
| 编辑作业设置                           |                  |            |                  | x          | x                    |
| 修改权限                          |                  |            |                  | x          | x                    |
| 删除作业                                  |                  |            |                  | x          | x                    |
| 更改所有者                                |                  |            |                  |            | x                    |

> [!NOTE]
>
> * 作业的创建者拥有“是所有者”权限。
> * 一个作业不能有多个所有者。
> * 作业不能将组作为所有者。
> * 通过“立即运行”触发的作业会获得作业所有者的权限，而不是发出“立即运行”的用户的权限 。 例如，即使将作业 A 配置为只能在作业所有者（用户 A）访问的现有群集上运行，具有“可以管理运行”权限的用户（用户 B）也可以启动该作业的新运行。
> * 仅当你在作业上有“可以查看”或更高的权限时，才能查看笔记本运行结果。 即使已重命名、移动或删除作业笔记本，此操作也可以使作业访问控制保持不变。
> * 作业访问控制适用于在 Databricks 作业 UI 中显示及运行的作业。 它不适用于[笔记本工作流](../../notebooks/notebook-workflows.md)生成的运行或 [API 提交的运行](../../dev-tools/api/latest/jobs.md#jobsjobsservicesubmitrun)，这些运行的 ACL 与笔记本捆绑在一起。

## <a name="enable-jobs-access-control"></a>启用作业访问控制

1. 转到[管理控制台](../../administration-guide/admin-console.md)。
2. 选择“访问控制”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“访问控制”选项卡](../../_static/images/admin-settings/access-control-tab.png)

3. 单击“群集和作业访问控制”旁边的“启用”按钮 。

   > [!div class="mx-imgBorder"]
   > ![启用访问控制](../../_static/images/access-control/cluster-and-jobs-acls.png)

4. 单击“确认”以确认更改。

## <a name="configure-job-permissions"></a>配置作业权限

> [!NOTE]
>
> 此部分介绍如何使用 UI 来管理权限。 你还可以使用[权限 API](../../_static/api-refs/permissions-azure.yaml)。

你必须具有“可以管理”或“是所有者”权限 。

1. 转到作业的[详细信息](../../jobs.md#job-details)页。
2. 单击“高级”。

   > [!div class="mx-imgBorder"]
   > ![高级](../../_static/images/access-control/job-advanced.png)

3. 单击“权限”旁边的“编辑”链接 。

   > [!div class="mx-imgBorder"]
   > ![编辑作业权限](../../_static/images/access-control/job-permissions.png)

4. 在弹出对话框中，通过用户名旁边的下拉菜单分配作业权限。

   > [!div class="mx-imgBorder"]
   > ![分配作业权限](../../_static/images/access-control/job-manage-acls.png)

5. 单击 **“保存更改”** 。