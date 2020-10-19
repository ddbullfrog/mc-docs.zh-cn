---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/14/2020
title: 群集访问控制 - Azure Databricks
description: 了解如何控制对 Azure Databricks 群集的访问。
ms.openlocfilehash: 32e3e8f6dba0975d03527792ed374a1295dd41ec
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937710"
---
# <a name="cluster-access-control"></a>群集访问控制

> [!NOTE]
>
> 访问控制仅在 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中提供。

默认情况下，除非管理员启用群集访问控制，否则所有用户均可创建和修改[群集](../../clusters/index.md)。 使用群集访问控制，用户的操作能力取决于权限。 本文介绍权限。

Azure Databricks 管理员必须先为工作区启用群集访问控制，然后你才能使用该控制。 请参阅[为工作区启用群集访问控制](../../administration-guide/access-control/cluster-acl.md)。

## <a name="types-of-permissions"></a>权限的类型

你可以配置两种类型的群集权限：

* “允许创建群集”权限控制你创建群集的能力。
* 群集级别权限控制你使用和修改特定群集的能力。

启用了群集访问控制时：

* 管理员可以对是否允许用户创建群集进行配置。
* 任何具有群集的“可管理”权限的用户都可以对是否允许用户附加到该群集、重启该群集、重设该群集大小和管理该群集进行配置。

## <a name="cluster-level-permissions"></a>群集级别权限

群集权限级别有四个：“无权限”、“可附加到”、“可重启”和“可管理”   。 该表列出了每个权限赋予用户的能力。

| 能力                          | 无权限    | 可附加到        | 可重启        | 可管理         |
|----------------------------------|-------------------|----------------------|--------------------|--------------------|
| 将笔记本附加到群集       |                   | x                    | x                  | x                  |
| 查看 Spark UI                    |                   | x                    | x                  | x                  |
| 查看群集指标             |                   | x                    | x                  | x                  |
| 终止群集                |                   |                      | x                  | x                  |
| 启动群集                    |                   |                      | x                  | x                  |
| 重启群集                  |                   |                      | x                  | x                  |
| 编辑群集                     |                   |                      |                    | x                  |
| 将库附加到群集        |                   |                      |                    | x                  |
| 调整群集大小                   |                   |                      |                    | x                  |
| 修改权限               |                   |                      |                    | x                  |

> [!NOTE]
>
> 你对自己创建的任何群集都具有“可管理”权限。

## <a name="configure-cluster-level-permissions"></a>配置群集级别权限

> [!NOTE]
>
> 此部分介绍如何使用 UI 来管理权限。 你还可以使用[权限 API](../../_static/api-refs/permissions-azure.yaml)。

群集访问控制必须[已启用](../../administration-guide/access-control/cluster-acl.md#cluster-acl-enable)，并且你必须具有针对群集的“可管理”权限。

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击现有群集的“操作”列下的 ![“权限”图标](../../_static/images/access-control/permissions-icon.png) 图标。

   > [!div class="mx-imgBorder"]
   > ![ClusterACLsButton](../../_static/images/access-control/cluster-acls-button.png)

3. 在“<cluster name> 的权限设置”对话框中，你可以：
   * 从“添加用户和组”下拉列表中选择用户和组，并为其分配权限级别。
   * 使用用户或组名称旁边的下拉菜单，为已添加的用户和组更新群集权限。

   > [!div class="mx-imgBorder"]
   > ![IndvClusterACLs](../../_static/images/access-control/user-cluster-acl.png)

4. 单击“Done”（完成） 。

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