---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 11/22/2019
title: 元存储 - Azure Databricks
description: 了解如何在 Azure Databricks 中使用外部元存储。
ms.openlocfilehash: 3935064f1658c2eedfc6b4845efb7748be1d0070
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121855"
---
# <a name="metastores"></a>元存储

每个 Azure Databricks 部署都有一个中心 Hive 元存储，供所有需要保存表元数据的群集访问。 可以选择使用现有的外部 Hive 元存储实例，而不使用 Azure Databricks Hive 元存储。

* [外部 Apache Hive 元存储](external-hive-metastore.md)