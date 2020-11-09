---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/08/2020
title: 管理数据上传 - Azure Databricks
description: 了解如何启用和禁用使用用户界面将数据上传到 Databricks 文件系统的功能。
ms.openlocfilehash: 7dc7383007d8c53346758fa3dd942e4ac9e9b6f8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106773"
---
# <a name="manage-data-upload"></a><a id="disable-ui-upload"> </a><a id="manage-data-upload"> </a>管理数据上传

作为管理员用户，你可以管理用户将数据上传到 Databricks 文件系统 (DBFS) 的权限，如下所示：

1. 转到[管理控制台](../admin-console.md)。
2. 单击“高级”  选项卡。
3. 单击“使用 UI 上传数据”右侧的“禁用”或“启用”按钮。 默认情况下启用此标志。

此设置不会控制通过特定方式（例如通过 DBFS 命令行界面）对 Databricks 文件系统进行的[编程访问](../../data/databricks-file-system.md#access-dbfs)。