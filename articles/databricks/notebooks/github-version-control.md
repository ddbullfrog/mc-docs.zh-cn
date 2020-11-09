---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: GitHub 版本控制 - Azure Databricks
description: 了解如何通过 UI 使用 GitHub 为笔记本设置版本控制。
ms.openlocfilehash: 3d9e3cc99bce0102a42b070e4a66d73e4309534d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106747"
---
# <a name="github-version-control"></a>GitHub 版本控制

本文介绍如何通过 UI 使用 GitHub 为笔记本设置版本控制。 本文档介绍如何通过 UI 设置 GitHub 集成，但你也可以使用 [Databricks CLI](../dev-tools/cli/index.md) 或[工作区 API](../dev-tools/api/latest/workspace.md) 来导入和导出笔记本，并使用 GitHub 工具管理笔记本版本。

## <a name="enable-and-disable-git-versioning"></a>启用和禁用 Git 版本控制

默认情况下启用版本控制。 若要切换此设置，请参阅[管理 Git 中的笔记本版本控制功能](../administration-guide/workspace/notebooks.md#manage-git-versioning)。 如果禁用 Git 版本控制，则“用户设置”屏幕中的“Git 集成”选项卡将不可用 。

## <a name="configure-version-control"></a>配置版本控制

配置版本控制需要在版本控制提供程序中创建访问凭据，然后将其添加到 Azure Databricks。

### <a name="get-an-access-token"></a>获取访问令牌

访问 GitHub 并创建允许访问存储库的个人访问令牌：

1. 打开 GitHub 右上角 Gravitar 旁的菜单，然后选择“设置”。
2. 单击“开发人员设置”。
3. 单击“个人访问令牌”选项卡。
4. 单击“生成新令牌”按钮。
5. 输入令牌说明。
6. 选择“存储库”权限，然后单击“生成令牌”按钮 。

   > [!div class="mx-imgBorder"]
   > ![生成 GitHub 令牌](../_static/images/version-control/github-newtoken.png)

7. 将令牌复制到剪贴板。 在下一步中，将此令牌输入 Azure Databricks。

若要详细了解如何创建个人访问令牌，请参阅 [GitHub 文档](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)。

### <a name="save-your-access-token-to-azure-databricks"></a>将访问令牌保存到 Azure Databricks

1. 单击屏幕右上方的用户图标![帐户图标](../_static/images/account-settings/account-icon.png)，然后选择“用户设置”。

   > [!div class="mx-imgBorder"]
   > ![AccountSettings](../_static/images/account-settings/user-settings.png)

2. 单击“Git 集成”选项卡。
3. 如果之前输入过凭据，请单击“更改令牌或应用密码”按钮。
4. 在“Git 提供程序”下拉列表中，选择“GitHub”。

   > [!div class="mx-imgBorder"]
   > ![选择“GitHub”Git 提供程序](../_static/images/version-control/github-provider-credentials.png)

5. 将令牌粘贴到“令牌或应用密码”字段，然后单击“保存” 。

## <a name="work-with-notebook-revisions"></a><a id="git-notebook"> </a><a id="work-with-notebook-revisions"> </a>使用笔记本修订版本

可以在“历史记录”面板中使用笔记本修订版本。 单击笔记本右上角的“修订版本历史记录”，以打开“历史记录”面板。

> [!div class="mx-imgBorder"]
> ![修订版本历史记录](../_static/images/version-control/revision-history-open.png)

> [!NOTE]
>
> “历史记录”面板处于打开状态时，无法修改笔记本。

### <a name="link-a-notebook-to-github"></a>将笔记本链接到 GitHub

1. 打开“历史记录”面板。 Git 状态栏显示“Git：未链接”。

   > [!div class="mx-imgBorder"]
   > ![Git 状态栏](../_static/images/version-control/git-not-linked.png)

2. 单击“Git：未链接”。

   此时将显示“Git 首选项”对话框。 第一次打开笔记本时，“状态”为“未链接”，因为笔记本不在 GitHub 中。

   > [!div class="mx-imgBorder"]
   > ![Git 首选项](../_static/images/version-control/git-new-link.png)

3. 在“状态”字段中，单击“链接”。
4. 在“链接”字段中，粘贴 GitHub 存储库的 URL。
5. 单击“分支”下拉列表，然后选择分支或键入新分支的名称。
6. 在“Git 存储库中的路径”字段中，指定文件在存储库中的存储位置。

   Python 笔记本具有推荐的默认文件扩展名 `.py`。 如果使用 `.ipynb`，则笔记本将以 iPython 笔记本格式保存。 如果文件已存在于 GitHub 中，则可以直接复制并粘贴文件的 URL。

7. 单击“保存”即可完成对笔记本的链接。 如果此文件之前不存在，则会显示一个提示，其中包含“将此文件保存到 GitHub 存储库”选项。
8. 键入消息，然后单击“保存”。

### <a name="save-a-notebook-to-github"></a>将笔记本保存到 GitHub

尽管对笔记本所做的更改会自动保存到 Azure Databricks 修订版本历史记录，但更改不会自动保存到 GitHub。

1. 打开“历史记录”面板。

   > [!div class="mx-imgBorder"]
   > ![“历史记录”面板](../_static/images/version-control/save-now.png)

2. 单击“立即保存”，将笔记本保存到 GitHub。 此时将显示“保存笔记本修订版本”对话框。
3. 根据需要，输入一条消息来描述所做的更改。
4. 确保选中“另提交到 Git”。

   > [!div class="mx-imgBorder"]
   > ![保存修订版本](../_static/images/version-control/save-revision.png)

5. 单击“保存”  。

### <a name="revert-or-update-a-notebook-to-a-version-from-github"></a>将笔记本还原或更新为 GitHub 中的版本

链接笔记本后，每次重新打开“历史记录”面板时，Azure Databricks 都会将历史记录与 Git 同步。 同步到 Git 的版本将提交哈希作为条目的一部分。

1. 打开“历史记录”面板。

   > [!div class="mx-imgBorder"]
   > ![“历史记录”面板](../_static/images/version-control/history.png)

2. 在“历史记录”面板中选择一个条目。 Azure Databricks 会显示该版本。
3. 单击“还原此版本”。
4. 单击“确认”，以确认想要还原该版本。

### <a name="unlink-a-notebook"></a>取消链接笔记本

1. 打开“历史记录”面板。
2. Git 状态栏显示“Git：已同步”。

   > [!div class="mx-imgBorder"]
   > ![Git 状态](../_static/images/version-control/save-now.png)

3. 单击“Git：已同步”。

   > [!div class="mx-imgBorder"]
   > ![Git 首选项](../_static/images/version-control/git-unlink.png)

4. 在“Git 首选项”对话框中，单击“取消链接”。
5. 单击“保存”  。
6. 单击“确认”，以确认想要通过版本控制取消链接笔记本。

### <a name="branch-support"></a>分支支持

可以使用存储库的任何分支，并在 Azure Databricks 内新建分支。

#### <a name="create-a-branch"></a>创建分支

1. 打开“历史记录”面板。
2. 单击 Git 状态栏以打开 GitHub 面板。
3. 单击“分支”下拉菜单。
4. 输入分支名。

   > [!div class="mx-imgBorder"]
   > ![创建分支](../_static/images/version-control/github-branch-create.png)

5. 选择下拉菜单底部的“创建分支”选项。 父分支已指出。 始终从当前选定的分支处创建分支。

#### <a name="create-a-pull-request"></a>创建拉取请求

1. 打开“历史记录”面板。
2. 单击 Git 状态栏以打开 GitHub 面板。

   > [!div class="mx-imgBorder"]
   > ![Git 状态](../_static/images/version-control/git-create-pr.png)

3. 单击“创建 PR”。 GitHub 打开分支的拉取请求页。

#### <a name="rebase-a-branch"></a>分支变基

你还可以在 Azure Databricks 内进行分支变基。 如果父分支中有新的提交，则会显示“变基”链接。 仅支持在父存储库的默认分支之上进行变基。

> [!div class="mx-imgBorder"]
> ![变基](../_static/images/version-control/github-rebase.png)

例如，假设你正在使用 `databricks/reference-apps`。 可以为其创建指向自己帐户的分支（例如 `brkyvz`），然后开始使用名为 `my-branch` 的分支。 如果将新的更新推送到 `databricks:master`，则显示 `Rebase` 按钮，并且可以将更改拉取到分支 `brkyvz:my-branch` 中。

在 Azure Databricks 中，变基的工作原理略有不同。 假设分支结构如下：

> [!div class="mx-imgBorder"]
> ![变基前的分支结构](../_static/images/version-control/rebase-before.png)

变基后，分支结构将如下所示：

> [!div class="mx-imgBorder"]
> ![变基后的分支结构](../_static/images/version-control/rebase-after.png)

此时不同之处在于，提交 C5 和 C6 不在 C4 之上应用， 而是显示为笔记本中的本地更改。 所有合并冲突将如下所示：

> [!div class="mx-imgBorder"]
> ![合并冲突](../_static/images/version-control/merge-conflict.png)

然后，可以使用“立即保存”按钮再次提交到 GitHub。

**如果有人从我刚变基的分支处创建分支，会发生什么？**

如果你的分支（例如 `branch-a`）是另一个分支 (`branch-b`) 的基，然后你进行了变基，则不必担心！ 用户也对 `branch-b` 进行变基后，所有问题都会迎刃而解。在这种情况下，最好对不同的笔记本使用不同的分支。

#### <a name="best-practices-for-code-reviews"></a>代码评审的最佳做法

Azure Databricks 支持 Git 分支。

* 可以将笔记本链接到自己的分支，然后选择分支。
* 建议为每个笔记本使用单独的分支。
* 对更改满意后，可以点击“Git 首选项”对话框中的“创建 PR”链接，转到 GitHub 的拉取请求页面。
* 仅在不使用父存储库的默认分支时，才会显示“创建 PR”链接。

## <a name="github-enterprise"></a>GitHub Enterprise

> [!IMPORTANT]
>
> 不支持与 GitHub Enterprise Server 集成。 但可以使用[工作区 API](../dev-tools/api/latest/workspace.md) 在 GitHub Enterprise Server 中以编程方式创建笔记本并管理代码库。

## <a name="troubleshooting"></a>疑难解答

如果收到与同步 GitHub 历史记录有关的错误，请验证以下内容：

1. 已初始化 GitHub 中的存储库，并且该存储库不为空。 试用输入的 URL，并验证其是否会转到 GitHub 存储库。
2. 个人访问令牌有效。
3. 如果存储库是专用的，则必须至少具有对存储库的读取级别权限（通过 GitHub）。