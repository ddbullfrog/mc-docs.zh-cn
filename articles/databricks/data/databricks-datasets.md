---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: Azure Databricks 数据集 - Azure Databricks
description: 了解如何浏览装载到 Databricks 文件系统的测试数据集。
ms.openlocfilehash: c6e562b71bbb8faa0e06f0a4802a719006c7c25e
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121954"
---
# <a name="azure-databricks-datasets"></a><a id="azure-databricks-datasets"> </a><a id="databricks-datasets"> </a>Azure Databricks 数据集

Azure Databricks 包括装载到 [Databricks 文件系统 (DBFS)](databricks-file-system.md) 的各种数据集，你可以使用这些数据集来了解 Apache Spark 或测试算法。 这些数据集遍布在所有文档页面中。

若要浏览这些文件，可以使用 [Databricks 实用工具](../dev-tools/databricks-utils.md)。 下面是可以用来列出所有 Databricks 数据集的代码片段。

```python
display(dbutils.fs.ls("/databricks-datasets"))
```

可以打印出任何数据集的 `README`，以获取其详细信息。

```python
with open("/dbfs/databricks-datasets/README.md") as f:
    x = ''.join(f.readlines())

print(x)
```