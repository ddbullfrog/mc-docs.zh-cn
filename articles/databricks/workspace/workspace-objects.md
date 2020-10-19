---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: 使用工作区对象 - Azure Databricks
description: 了解如何使用文件夹和其他 Azure Databricks 工作区对象。
ms.openlocfilehash: 8e4677ff592e958b2b149fcd26605e5475e2abec
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937786"
---
# <a name="work-with-workspace-objects"></a>使用工作区对象

本文介绍如何使用文件夹和其他工作区对象。

## <a name="folders"></a>文件夹

文件夹包含工作区中的所有静态资产：笔记本、库、试验和其他文件夹。
图标指示文件夹中包含的对象的类型。 单击文件夹名称以打开或关闭该文件夹，并查看其内容。

> [!div class="mx-imgBorder"]
> ![打开文件夹](../_static/images/workspace/folder-open.png)

若要对文件夹执行操作，请单击 ![文件夹右侧的](../_static/images/down-caret.png) 向下插入符，然后选择菜单项。

> [!div class="mx-imgBorder"]
> ![文件夹菜单](../_static/images/workspace/folder-menu.png)

### <a name="special-folders"></a>特殊文件夹

Azure Databricks 工作区包括三个特殊文件夹：工作区、共享和用户。 无法重命名或移动特殊文件夹。

#### <a name="workspace-root-folder"></a>工作区根文件夹

若要导航到工作区根文件夹：

1. 安装完成后，单击 ![工作区图标](../_static/images/workspace/workspace-icon.png) 或 ![主页图标](../_static/images/workspace/home-icon.png) 来折叠导航窗格。
2. 安装完成后，单击 ![向左滚动图标](../_static/images/workspace/scroll-left-icon.png) 来折叠导航窗格。

工作区根文件夹是组织中所有 Azure Databricks 静态资产的容器。

![工作区根目录](../_static/images/workspace/workspace-folder.png)

在工作区根文件夹内：

* ![共享图标](../_static/images/workspace/shared-icon.png)“共享”用于在组织中共享对象。 所有用户都对“共享”中的所有对象拥有完全权限。
* ![用户图标](../_static/images/workspace/users-icon.png)“用户”包含每个用户的文件夹。

默认情况下，工作区根文件夹及其包含的所有对象对所有用户都可用。 你可以通过[启用工作区访问控制](../administration-guide/access-control/workspace-acl.md#enable-workspace-acl)，并设置[权限](../security/access-control/workspace-acl.md#permissions)来控制谁可以管理和访问对象。

若要在所有文件夹中按字母顺序或按类型对所有对象进行排序，请单击工作区文件夹右侧的![向下插入符](../_static/images/down-caret.png)，并选择“排序> [字母顺序 | 类型]”：

![工作区排序](../_static/images/workspace/workspace-sort.png)

#### <a name="user-home-folders"></a><a id="home-folder"> </a><a id="user-home-folders"> </a>用户主文件夹

每个用户在其笔记本和库中都有一个主文件夹：

![工作区](../_static/images/workspace/workspace.png) > ![主文件夹](../_static/images/workspace/home-folder.png)

如果已启用工作区访问控制，则默认情况下，此文件夹中的对象是该用户的专用对象。

> [!NOTE]
>
> 如果[删除用户](../administration-guide/users-groups/users.md#remove-user)，则会保留用户的主文件夹。

## <a name="workspace-object-operations"></a><a id="objects"> </a><a id="workspace-object-operations"> </a>工作区对象操作

工作区根文件夹中存储的对象是[文件夹](#folders)、[笔记本](workspace-assets.md#ws-notebooks)、[库](workspace-assets.md#ws-libraries)和[试验](../applications/mlflow/tracking.md#mlflow-experiments)。 若要对工作区对象执行操作，请右键单击该对象，或单击对象右侧的![菜单下拉列表](../_static/images/menu-dropdown.png)。

> [!div class="mx-imgBorder"]
> ![对象菜单](../_static/images/workspace/object-menu.png)

从下拉菜单中，你可以：

* 如果对象是文件夹：
  * 创建[笔记本](../notebooks/index.md)、[库](../libraries/index.md)、[MLflow 试验](../applications/mlflow/tracking.md#mlflow-experiments)或文件夹。
  * 导入 [Databricks 存档](../notebooks/notebooks-manage.md#databricks-archive)。
* 克隆对象。
* 重命名对象。
* 将对象移动到另一文件夹。
* 将对象移动到回收站。 请参阅[删除对象](#delete-object)。
* 将文件夹或笔记本导出为 Databricks 存档。
* 如果对象是笔记本，请复制笔记本的文件路径。
* 如果已启用[工作区访问控制](../security/access-control/workspace-acl.md)，请在对象上设置权限。

### <a name="search-workspace-for-an-object"></a><a id="search-objects"> </a><a id="search-workspace-for-an-object"> </a>对象的搜索工作区

若要在工作区中搜索对象，请单击侧边栏中的搜索图标![搜索图标](../_static/images/search/search-icon.png)，然后在“搜索工作区”字段中键入搜索字符串。 键入时，会列出其名称包含搜索字符串的对象。

> [!div class="mx-imgBorder"]
> ![搜索字符串匹配项](../_static/images/workspace/search-ex.png)

### <a name="access-recently-used-objects"></a><a id="access-recently-used-objects"> </a><a id="recent_objects"> </a>访问最近使用过的对象

可以通过单击“最近使用”图标来访问最近使用过的对象 ![“最近使用”图标](../_static/images/workspace/recents-icon.png) 位于工作区登陆页上的侧边栏或“最近使用”列中。

> [!NOTE]
>
> 删除浏览器缓存和 cookie 后，会清除“最近使用”列表。

### <a name="move-an-object"></a><a id="move-an-object"> </a><a id="move-object"> </a>移动对象

若要移动对象，可以拖放对象，或者单击![菜单下拉列表](../_static/images/menu-dropdown.png)或对象右侧的![向下插入符](../_static/images/down-caret.png)，然后选择“移动”：

> [!div class="mx-imgBorder"]
> ![移动对象](../_static/images/workspace/move-drop-down.png)

若要将文件夹中的所有对象移动到其他文件夹，请在源文件夹上选择“移动”操作，然后选择“移动 <folder-name> 内的所有项，而不是文件夹本身”复选框  。

> [!div class="mx-imgBorder"]
> ![移动文件夹内容](../_static/images/workspace/move-contents-check-box.png)

### <a name="delete-an-object"></a><a id="delete-an-object"> </a><a id="delete-object"> </a>删除对象

若要删除文件夹、笔记本、库或试验，请单击![菜单下拉列表](../_static/images/menu-dropdown.png)或对象右侧的![向下插入符](../_static/images/down-caret.png)，然后选择“移至回收站”。 30 天后，回收站文件夹会自动清空（清除）。

你可以通过选择对象右侧的![菜单下拉列表](../_static/images/menu-dropdown.png)然后选择“立即删除”，以永久删除回收站中的对象。

> [!div class="mx-imgBorder"]
> ![立即删除](../_static/images/workspace/trash-delete.png)

你可以通过选择回收站文件夹右侧的![菜单下拉列表](../_static/images/menu-dropdown.png)然后选择“清空回收站”，以永久删除回收站中的对象。

> [!div class="mx-imgBorder"]
> ![清空回收站](../_static/images/workspace/empty-trash.png)

### <a name="restore-an-object"></a>还原对象

通过将对象从回收站文件夹拖到其他文件夹 ![回收站](../_static/images/workspace/trash-icon.png) 来还原对象。