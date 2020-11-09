---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: 包单元 - Azure Databricks
description: 了解如何在包单元中定义类，以便在 Apache Spark 和跨笔记本会话中可靠地使用笔记本中定义的自定义 Scala 类和对象。
ms.openlocfilehash: 699ab842874d21be4abb9f5cd3b564f2c7f8ce5a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106648"
---
# <a name="package-cells"></a>包单元

若要在 Spark 中以及跨笔记本会话中可靠地使用笔记本中定义的自定义 Scala 类和对象，应在包单元中定义类。 “包单元”是在运行时编译的单元。 包单元对于笔记本的其余部分不可见。 可以将其视为单独的 Scala 文件。  只有 `class` 和 `object` 定义可以放入包单元。 不能放入任何值、变量或函数定义。

以下笔记本显示了如果不使用包单元会发生什么情况，并提供了一些示例、注意事项和最佳做法。

## <a name="package-cells-notebook"></a>包单元笔记本

[获取笔记本](../_static/notebooks/package-cells.html)