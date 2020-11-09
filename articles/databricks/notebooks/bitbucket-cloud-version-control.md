---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: Bitbucket 云版本控制 - Azure Databricks
description: 了解如何通过 UI 使用 Bitbucket 云为笔记本设置版本控制。
ms.openlocfilehash: 9ff4b549fda550d5842be72cc9fe1f93700a3c45
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106722"
---
# <a name="bitbucket-cloud-version-control"></a><a id="bitbucket-cloud"> </a><a id="bitbucket-cloud-version-control"> </a>Bitbucket 云版本控制

本指南介绍了如何通过 UI 使用 Bitbucket 云为笔记本设置版本控制。 虽然本文档介绍的是如何通过 UI 设置 Bitbucket 云集成，但你也可以使用 [Databricks CLI](../dev-tools/cli/index.md) 或[工作区 API](../dev-tools/api/latest/workspace.md) 来导入和导出笔记本，并使用 Bitbucket 工具管理笔记本版本。

## <a name="enable-and-disable-git-versioning"></a>启用和禁用 Git 版本控制

默认情况下会启用版本控制。 若要切换此设置，请参阅[管理 Git 中的笔记本版本控制功能](../administration-guide/workspace/notebooks.md#manage-git-versioning)。 如果禁用 Git 版本控制，则“用户设置”屏幕中的“Git 集成”选项卡将不可见 。

## <a name="configure-version-control"></a>配置版本控制

配置版本控制需要在版本控制提供程序中创建访问凭据，然后将这些凭据添加到 Azure Databricks。

### <a name="get-an-app-password"></a>获取应用密码

1. 转到 Bitbucket 云，创建允许访问存储库的应用密码。 请参阅 [Bitbucket 云文档](https://confluence.atlassian.com/bitbucket/app-passwords-828781300.html)。
2. 记录密码。 在下一步中，在 Azure Databricks 中输入此密码。

### <a name="save-your-app-password-and-username-to-azure-databricks"></a>将你的应用密码和用户名保存到 Azure Databricks

1. 单击屏幕右上方的用户图标 ![帐户图标](../_static/images/account-settings/account-icon.png)，然后选择“用户设置”。

   > [!div class="mx-imgBorder"]
   > 帐户设置![](../_static/images/account-settings/user-settings.png)

2. 单击“Git 集成”选项卡。
3. 如果你之前输入过凭据，请单击“更改令牌或应用密码”按钮。
4. 在 Git 提供程序下拉列表中，选择“Bitbucket 云”。

   > [!div class="mx-imgBorder"]
   > ![Bitbucket 云 GitHub 提供程序](../_static/images/version-control/bb-cloud-provider-credentials.png)

5. 将密码和用户名粘贴到相应的字段中，然后单击“保存”。

## <a name="work-with-notebook-revisions"></a>使用笔记本修订版

可以在“历史记录”面板中使用笔记本修订版本。 单击笔记本右上角的“修订历史记录”，以打开历史记录面板。

> [!div class="mx-imgBorder"]
> ![修订历史记录](../_static/images/version-control/revision-history-open.png)

> [!NOTE]
>
> “历史记录”面板处于打开状态时，无法修改笔记本。

### <a name="link-a-notebook-to-bitbucket-cloud"></a>将笔记本链接到 Bitbucket 云

1. 打开“历史记录”面板。 Git 状态栏显示“Git:未链接”。

   > [!div class="mx-imgBorder"]
   > ![“历史记录”面板](../_static/images/version-control/git-not-linked.png)

2. 单击“Git:未链接”。

   此时会显示“Git 首选项”对话框。 第一次打开笔记本时，“状态”为“未链接”，因为笔记本不在 Bitbucket 云中。

   > [!div class="mx-imgBorder"]
   > ![Git 首选项](../_static/images/version-control/git-new-link.png)

3. 在“状态”字段中，单击“链接”。
4. 在“链接”字段中，粘贴 Bitbucket 云存储库的 URL。
5. 单击“分支”下拉列表，选择一个分支。
6. 在“Git 存储库中的路径”字段中，指定文件在存储库中的存储位置。

   Python 笔记本具有建议的默认文件扩展名 `.py`。 如果使用 `.ipynb`，则笔记本会以 iPython 笔记本格式保存。 如果文件已存在于 Bitbucket 云中，则可以直接复制并粘贴文件的 URL。

7. 单击“保存”即可完成对笔记本的链接。 如果此文件之前不存在，则会显示一个提示，其中包含“将此文件保存到 Bitbucket 云存储库”选项。
8. 键入一条消息，然后单击“保存”。

### <a name="save-a-notebook-to-bitbucket-cloud"></a>将笔记本保存到 Bitbucket 云

尽管对笔记本所做的更改会自动保存到 Azure Databricks 修订历史记录，但更改不会自动保存到 Bitbucket 云。

1. 打开“历史记录”面板。

   > [!div class="mx-imgBorder"]
   > ![“历史记录”面板](../_static/images/version-control/save-now.png)

2. 单击“立即保存”，将笔记本保存到 Bitbucket 云。 此时会显示“保存笔记本修订版本”对话框。
3. 根据需要，输入一条消息以对更改进行说明。
4. 确保选中“另提交到 Git”。

   > [!div class="mx-imgBorder"]
   > ![保存修订版本](../_static/images/version-control/save-revision.png)

5. 单击“保存”  。

### <a name="revert-or-update-a-notebook-to-a-version-from-bitbucket-cloud"></a>将笔记本还原或更新为 Bitbucket 云中的版本

链接笔记本后，每次重新打开“历史记录”面板时，Azure Databricks 都会将历史记录与 Git 同步。 同步到 Git 的版本将提交哈希作为条目的一部分。

1. 打开“历史记录”面板。

   > [!div class="mx-imgBorder"]
   > ![“历史记录”面板](../_static/images/version-control/history.png)

2. 在“历史记录”面板中选择一个条目。 Azure Databricks 会显示该版本。
3. 单击“还原此版本”。
4. 单击“确认”，以确认是否要还原该版本。

### <a name="unlink-a-notebook"></a>取消链接笔记本

1. 打开“历史记录”面板。
2. Git 状态栏显示“Git: **已同步”。**

   > [!div class="mx-imgBorder"]
   > ![Git 状态](../_static/images/version-control/save-now.png)

3. 单击“Git: **已同步”。**

   > [!div class="mx-imgBorder"]
   > ![Git 首选项](../_static/images/version-control/git-unlink.png)

4. 在“Git 首选项”对话框中，单击“取消链接”。
5. 单击“保存”  。
6. 单击“确认”，以确认是否要断开笔记本与版本控制的链接。

### <a name="create-a-pull-request"></a>创建拉取请求

1. 打开“历史记录”面板。
2. 单击 Git 状态栏以打开“Git 首选项”对话框。

   > [!div class="mx-imgBorder"]
   > ![Git 首选项](../_static/images/version-control/git-create-pr.png)

3. 单击“创建 PR”。 Bitbucket 云会打开到分支的拉取请求页。

## <a name="best-practice-for-code-reviews"></a>代码评审的最佳做法

Azure Databricks 支持进行 Git 分支。

* 可以将笔记本链接到你自己的分支，然后选择一个分支。
* 建议为每个笔记本使用单独的分支。
* 对更改满意后，可以点击“Git 首选项”对话框中的“创建 PR”链接，转到 Bitbucket 云的拉取请求页面。
* 仅在不使用父存储库的默认分支时，才会显示“创建 PR”链接。

## <a name="bitbucket-server"></a>Bitbucket 服务器

> [!IMPORTANT]
>
> 不支持 Bitbucket 服务器集成。 但是，可以使用[工作区 API](../dev-tools/api/latest/workspace.md) 在 Bitbucket 服务器中以编程方式创建笔记本并管理代码库。

## <a name="troubleshooting"></a>疑难解答

如果收到与 Bitbucket 云历史记录同步相关的错误，请验证以下事项：

1. 你已初始化 Bitbucket 云中的存储库，该存储库不为空。 试用你输入的 URL，验证它是否转发到你的 Bitbucket 云存储库。
2. 你的应用密码处于活动状态，你的用户名正确。
3. 如果存储库是专用的，你应该会在存储库上获得读取和写入访问权限（通过 Bitbucket 云获得）。