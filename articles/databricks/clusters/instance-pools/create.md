---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 创建池 - Azure Databricks
description: 了解如何创建 Azure Databricks 池。
ms.openlocfilehash: 670a71567bf1aa8b5058d579c8ef42ee67fc561a
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121798"
---
# <a name="create-a-pool"></a><a id="create-a-pool"> </a><a id="instance-pools-create"> </a>创建池

> [!IMPORTANT]
>
> 你必须有权创建池；请参阅[池访问控制](../../security/access-control/pool-acl.md)。

本文介绍如何使用 UI 创建池。

* 若要了解如何使用 Databricks CLI 创建池，请参阅[实例池 CLI](../../dev-tools/cli/instance-pools-cli.md)。
* 若要了解如何使用 REST API 创建池，请参阅[实例池 API](../../dev-tools/api/latest/instance-pools.md)。

若要创建池，请执行以下操作：

1. 单击“群集”图标 ![“群集”图标](../../_static/images/clusters/clusters-icon.png) （在边栏中）。
2. 单击“池”选项卡。

   > [!div class="mx-imgBorder"]
   > ![“池”选项卡](../../_static/images/instance-pools/tab.png)

3. 单击页面顶部的“创建池”按钮。

   > [!div class="mx-imgBorder"]
   > ![创建池](../../_static/images/instance-pools/create.png)

4. 指定[池配置](configure.md#instance-pool-configurations)。
5. 单击“创建”  按钮。

   > [!div class="mx-imgBorder"]
   > ![“创建”按钮](../../_static/images/instance-pools/create-detail.png)

你会注意到空闲实例处于挂起状态。 当这些实例不再挂起时，连接到该池的群集将启动得更快。

> [!div class="mx-imgBorder"]
> ![池详细信息](../../_static/images/instance-pools/pending.png)

若要使用 REST API 创建池，请参阅[实例池 API](../../dev-tools/api/latest/instance-pools.md) 文档。