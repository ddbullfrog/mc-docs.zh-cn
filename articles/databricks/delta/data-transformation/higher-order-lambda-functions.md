---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 高阶函数 - Azure Databricks
description: 了解 Azure Databricks 如何优化高阶函数和内置函数的性能。
ms.openlocfilehash: 80652ba14e35199a43777c2183d8c3f14ce19e97
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121904"
---
# <a name="higher-order-functions"></a><a id="higher-order-functions"> </a><a id="hof"> </a>高阶函数

Azure Databricks 提供了专用的基元，用于处理 Apache Spark SQL 中的数组；这使得使用数组变得更容易、更简洁，并省去了通常需要的大量样板代码。 基元围绕两个函数式编程构造：高阶函数和匿名 (lambda) 函数。 这两种函数协同工作，让你能够定义在 SQL 中操作数组的函数。 高阶函数采用一个数组，实现数组的处理方式以及计算结果。 它委托 lambda 函数处理数组中的每一项。

## <a name="introduction-to-higher-order-functions-notebook"></a>高阶函数笔记本简介

[获取笔记本](../../_static/notebooks/higher-order-functions.html)

## <a name="higher-order-functions-tutorial-python-notebook"></a>高阶函数教程 Python 笔记本

[获取笔记本](../../_static/notebooks/higher-order-functions-tutorial-python.html)

## <a name="apache-spark-built-in-functions"></a><a id="apache-spark-built-in-functions"> </a><a id="built-in-fns"> </a>Apache Spark 内置函数

Apache Spark 具有用于操作复杂类型（例如数组类型）的内置函数，包括高阶函数。

以下笔记本说明了 Apache Spark 内置函数。

### <a name="apache-spark-built-in-functions-notebook"></a>Apache Spark 内置函数笔记本

[获取笔记本](../../_static/notebooks/apache-spark-functions.html)