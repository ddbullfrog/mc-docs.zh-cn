---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 删除池 - Azure Databricks
description: 了解如何删除 Azure Databricks 池。
ms.openlocfilehash: f169b555e3194ef78466d33483c815517ea1839c
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121793"
---
# <a name="delete-a-pool"></a><a id="delete-a-pool"> </a><a id="instance-pool-delete"> </a>删除池

删除池会终止池的空闲实例并删除其配置。

> [!WARNING]
>
> 不能撤消此操作。

若要删除池，请单击[池](display.md#instance-pools-display)页上操作中的 ![“删除”图标](../../_static/images/clusters/delete-icon.png) 图标。

> [!div class="mx-imgBorder"]
> ![删除池](../../_static/images/instance-pools/delete-list.png)

> [!NOTE]
>
> * 附加到该池的正在运行的群集会继续运行，但是无法在重设大小或纵向扩展过程中分配实例。
> * 附加到该池的已终止群集将无法启动。

还可以调用[删除](../../dev-tools/api/latest/instance-pools.md#clusterinstancepoolservicedeleteinstancepool) API 终结点，以编程方式删除池。