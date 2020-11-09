---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 管理工作区存储 - Azure Databricks
description: 了解如何在 Azure Databricks 中清除工作区存储。
ms.openlocfilehash: bec2f3a9a44e7cc2e22e6088946a035142b58dbd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106767"
---
# <a name="manage-workspace-storage"></a>管理工作区存储

为了符合组织的隐私要求，有时可能需要清除已删除的对象，例如笔记本单元、整个笔记本、试验或群集日志。

作为管理员用户，你可以执行以下清除操作：

1. 转到[管理控制台](../admin-console.md)。
2. 单击“工作区存储”选项卡。

## <a name="purge-workspace"></a>清除工作区

你可以删除工作区对象，例如整个笔记本、单个笔记本单元、单个笔记本注释和试验，但它们是可恢复的。

若要永久清除已删除的工作区对象，请执行以下操作：

1. 单击 **清除** 按钮。

   > [!div class="mx-imgBorder"]
   > ![清除工作区](../../_static/images/admin-settings/purge-workspace.png)

2. 单击“是，清除”进行确认。

   > [!WARNING]
   >
   > 清除后，工作区对象将无法恢复。

## <a name="purge-notebook-revision-history"></a>清除笔记本修订历史记录

若要永久清除笔记本修订历史记录，请执行以下操作：

1. 在“时间范围”下拉列表中，选择要清除的时间范围：
2. 单击 **清除** 按钮。

   > [!div class="mx-imgBorder"]
   > ![清除笔记本修订历史记录](../../_static/images/admin-settings/purge-revision-history.png)

3. 单击“是，清除”进行确认。

> [!WARNING]
>
> 清除后，修订历史记录将无法恢复。

## <a name="purge-cluster-logs"></a>清除群集日志

若要为工作区中的所有群集永久清除 Spark 驱动程序日志和历史指标快照，请执行以下操作：

1. 单击 **清除** 按钮。

   > [!div class="mx-imgBorder"]
   > ![清除群集日志](../../_static/images/admin-settings/purge-cluster-logs.png)

2. 单击“是，清除”进行确认。

   > [!WARNING]
   >
   > 清除后，群集日志将无法恢复。