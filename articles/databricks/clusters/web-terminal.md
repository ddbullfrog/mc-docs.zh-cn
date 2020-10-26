---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: Web 终端 - Azure Databricks
description: 了解如何使用 Azure Databricks Web 终端。
ms.openlocfilehash: bb54ea8fdd099e5cdc7ab633acea1a8945471ba9
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121879"
---
# <a name="web-terminal"></a>Web 终端

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../release-notes/release-types.md)提供。

Azure Databricks Web 终端提供了一种便捷且高度交互的方式，使你可以在 Spark 驱动程序节点上运行 shell 命令并使用 Vim 或 Emacs 等编辑器。 一个群集上的多位用户可使用 Web 终端。 使用 Web 终端的示例包括监视资源使用情况和安装 Linux 包。

默认情况下，针对所有工作区用户禁用 Web 终端。

> [!WARNING]
>
> Azure Databricks 代理群集的 Spark 驱动程序上端口 7681 中的 Web 终端服务。 此 Web 代理仅用于 Web 终端。 如果在群集启动时该端口被占用，或者存在其他冲突，Web 终端可能无法按预期工作。 如果在端口 7681 上启动了其他 Web 服务，则群集用户可能会面临潜在的安全漏洞。 Databricks 和 Microsoft 均不负责因在群集上安装不受支持的软件而导致的任何问题。

## <a name="requirements"></a>要求

* Databricks Runtime 7.0 及更高版本.
* 群集上的[可附加到](../security/access-control/cluster-acl.md#cluster-level-permissions)权限。
* Azure Databricks 工作区必须[启用](../administration-guide/clusters/web-terminal.md) Web 终端。

## <a name="use-the-web-terminal"></a>使用 Web 终端

执行下列操作之一：

* 在群集详细信息页中，单击“应用”选项卡，然后单击“启动 Web 终端” 。

  > [!div class="mx-imgBorder"]
  > ![启动 Web 终端](../_static/images/clusters/web-terminal-launch.png)

* 在笔记本中，单击附加的群集下拉箭头，然后单击“终端”。

  > [!div class="mx-imgBorder"]
  > ![启动 Web 终端快捷方式](../_static/images/clusters/web-terminal-launch-shortcut.png)

此时会打开一个新的选项卡，其中包含 Web 终端 UI 和 Bash 提示，你可以在其中以根用户身份在群集驱动程序节点内运行命令。

> [!div class="mx-imgBorder"]
> ![Web 终端 UI](../_static/images/clusters/web-terminal-ui.png)

每个用户最多可以打开 100 个活动 Web 终端会话（选项卡）。 空闲的 Web 终端会话可能会超时，Web 终端 Web 应用程序将重新连接，从而生成新的 shell 进程。 如果要保留 Bash 会话，Databricks 建议使用 [tmux](https://www.man7.org/linux/man-pages/man1/tmux.1.html)。

## <a name="limitations"></a>限制

Azure Databricks Web 终端在以下群集类型中不可用：

* 已启用[表访问控制](../security/access-control/table-acls/index.md)或[凭据传递](../security/credential-passthrough/index.md)的高并发群集。
* 通过 `DISABLE_WEB_TERMINAL=true` 环境变量集启动的群集。