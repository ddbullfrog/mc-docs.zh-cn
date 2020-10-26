---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 编辑池 - Azure Databricks
description: 了解如何编辑 Azure Databricks 池。
ms.openlocfilehash: 52cd233546954338e2bc62a23a7726398637fed3
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121861"
---
# <a name="edit-a-pool"></a><a id="edit-a-pool"> </a><a id="instance-pools-edit"> </a>编辑池

可以从“池详细信息”页编辑池配置。

> [!div class="mx-imgBorder"]
> ![编辑池](../../_static/images/instance-pools/edit.png)

你可以编辑[池配置](configure.md#instance-pool-configurations)的子集；不可编辑的配置显示为灰色。

> [!div class="mx-imgBorder"]
> ![池配置](../../_static/images/instance-pools/edit-disabled-fields.png)

还可以调用[编辑](../../dev-tools/api/latest/instance-pools.md#clusterinstancepoolserviceeditinstancepool) API 以编程方式编辑池。

> [!NOTE]
>
> 附加到池的群集在编辑后将保持附加状态。