---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/12/2020
title: Azure DevOps Services 版本控制 - Azure Databricks
description: 了解如何将 Azure DevOps 设置为 Git 提供程序。
ms.openlocfilehash: 7ccdeea3bebea261eae2df24541c6977631a2d24
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106795"
---
# <a name="azure-devops-services-version-control"></a>Azure DevOps Services 版本控制

Azure DevOps 是一系列服务，这些服务为 DevOps 的五个核心做法提供端到端解决方案，这五个核心做法是：规划和跟踪、开发、生成和测试、交付，以及监视和操作。 本文介绍了如何将 Azure DevOps 设置为 Git 提供程序。

> [!NOTE]
>
> 有关名称从 Visual Studio Team Services 更改到 Azure DevOps 的信息，请参阅 [Visual Studio Team Services 现在是 Azure DevOps Services](/devops/user-guide/what-is-azure-devops?view=azure-devops#visual-studio-team-services-is-now-azure-devops-services)。

## <a name="enable-and-disable-git-versioning"></a>启用和禁用 Git 版本控制

在默认情况下，版本控制处于启用状态。 若要切换此设置，请参阅[管理 Git 中的笔记本版本控制功能](../administration-guide/workspace/notebooks.md#manage-git-versioning)。 如果禁用了 Git 版本控制，则“用户设置”屏幕中不会有“Git 集成”选项卡 。

## <a name="get-started"></a>入门

在使用 Azure Active Directory (Azure AD) 进行身份验证时，会自动使用 [Azure DevOps Services](/devops/?view=vsts) 完成身份验证。 Azure DevOps Services 组织必须链接到与 Databricks 相同的 Azure AD 租户。

在 Azure Databricks 中，在“用户设置”页上将 Git 提供程序设置为 Azure DevOps Services：

1. 单击屏幕右上方的“用户”图标 ![帐户图标](../_static/images/account-settings/account-icon.png)，然后选择“用户设置”。

   > [!div class="mx-imgBorder"]
   > 帐户设置![](../_static/images/account-settings/user-settings.png)

2. 单击“Git 集成”选项卡。
3. 将提供程序更改为 Azure DevOps Services。

   > [!div class="mx-imgBorder"]
   > ![Azure DevOps Services GitHub 提供程序](../_static/images/version-control/devops-provider-credentials.png)

## <a name="notebook-integration"></a>笔记本集成

笔记本与 Azure DevOps Services 的集成和笔记本与 GitHub 的集成是完全相同的。 请参阅[使用笔记本修订版本](github-version-control.md#git-notebook)，以详细了解如何使用 Git 来处理笔记本。

> [!TIP]
>
> 在“Git 首选项”中，使用 URL 方案 `https://dev.azure.com/<org>/<project>/_git/<repo>` 将 Azure DevOps 和 Azure Databricks 链接到同一个 Azure AD 租户。
>
>   ![Git 首选项](../_static/images/version-control/git-preferences-azuredevops.png)
>
> 如果你的 Azure DevOps 组织为 `org.visualstudio.com`，请在浏览器中打开 `dev.azure.com` 并导航到你的存储库。 从浏览器中复制 URL，并将该 URL 粘贴到“链接”字段中。

## <a name="troubleshooting"></a>疑难解答

**Databricks UI 中的“保存”按钮已灰显。**

[Visual Studio Team Services 已重命名为 Azure DevOps Services](/devops/user-guide/what-is-azure-devops?view=azure-devops#visual-studio-team-services-is-now-azure-devops-services)。 `https://<org>.visualstudio.com/<project>/_git/<repo>` 格式的原始 URL 在 Azure Databricks 笔记本中不起作用。

组织管理员可以从组织设置页自动更新 Azure DevOps Services 中的 URL。

或者，你也可以手动创建要与 Azure DevOps Services 同步的 Azure Databricks 笔记本中使用的新 URL 格式。 在 Azure Databricks 笔记本中，在“Git 首选项”对话框中的“链接”字段中输入新 URL。

旧 URL 格式：

`https://<org>.visualstudio.com/<project>/_git/<repo>`

新 URL 格式：

`https://dev.azure.com/<org>/<project>/_git/<repo>`