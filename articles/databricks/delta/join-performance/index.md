---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 优化联接性能 - Azure Databricks
description: 了解 Azure Databricks 上的 Delta Lake 如何优化联接性能。
ms.openlocfilehash: b92731b4d1005cc70480fdf2b7161abde74bbfa1
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121854"
---
# <a name="optimize-join-performance"></a><a id="join-performance"> </a><a id="optimize-join-performance"> </a>优化联接性能

Azure Databricks 上的 Delta Lake 优化了范围和倾斜联接。 范围联接优化要求基于查询模式进行优化，倾斜联接可以通过倾斜提示提高效率。 请参阅以下文章，了解如何对这些联接优化进行最佳利用：

* [范围联接优化](range-join.md)
* [倾斜联接优化](skew-join.md)