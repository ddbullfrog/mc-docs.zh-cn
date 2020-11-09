---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: 启用 Web 终端 - Azure Databricks
description: 了解如何启用和管理 Azure Databricks Web 终端。
ms.openlocfilehash: 4148a7c9554387cfc50da7757368e6e7386c9418
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106505"
---
# <a name="enable-web-terminal"></a>启用 Web 终端

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../release-notes/release-types.md)提供。

若要在群集上使用 [Web 终端](../../clusters/web-terminal.md)，工作区管理员必须按如下所示启用 Web 终端：

1. 转到[管理控制台](../admin-console.md)。
2. 单击“高级”  选项卡。
3. 单击“Web 终端”旁边的“启用”按钮 。
4. 刷新页面。

> [!NOTE]
>
> 在管理控制台中启用该功能后，你可能需要等待几分钟，以便配置值传播到所有群集。

## <a name="manage-the-web-terminal-per-cluster"></a>按群集管理 Web 终端

为了防止用户使用 Web 终端访问群集：

* 不要向任何用户授予“可附加到”权限。
* 若要启用笔记本访问而不是 Web 终端访问，请在群集配置中设置 `DISABLE_WEB_TERMINAL=true` 环境变量。

  > [!div class="mx-imgBorder"]
  > ![群集配置](../../_static/images/clusters/web-terminal-config.png)

## <a name="security-considerations"></a>安全注意事项

当你注销 Azure Databricks 工作区（或在工作区上禁用该功能）后，活动的 Web 终端会话将继续处于活动状态，直到刷新页面或页面超时为止。这是已知问题。

当群集所有者删除另一个用户对群集的“可附加到”权限时，该用户的活动 Web 终端会话将继续处于活动状态，直到刷新页面或由于不活动而导致页面超时。 这是一个已知问题。

可以通过重新启动受影响的群集来缓解这两个漏洞，这会强制 Web 终端会话终止。

## <a name="requirements"></a>要求

Azure Databricks Web 终端可在 Databricks Runtime 7.0 及更高版本中使用，但也有少数的例外情况，如[限制](../../clusters/web-terminal.md#limitations)中所述。