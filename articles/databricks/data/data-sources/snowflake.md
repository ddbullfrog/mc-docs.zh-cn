---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: Snowflake - Azure Databricks
description: 了解如何使用 Azure Databricks 在 Snowflake 上读取和写入数据。
ms.openlocfilehash: 74ed73a76b73f5eae93b14774a63dc5b93ad3cdf
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121912"
---
# <a name="snowflake"></a>Snowflake

[Snowflake](https://www.snowflake.com/) 是一种基于云的 SQL 数据仓库，侧重于卓越的性能、零优化、多样性的数据源和安全性。 本文介绍如何使用 Databricks Snowflake 连接器从 Snowflake 中读取数据并将数据写入到 Snowflake。

Azure Databricks 和 Snowflake 合作为 Azure Databricks 和 Snowflake 的客户带来一流的连接器体验，使你无需将库导入并加载到群集中，从而避免了版本冲突和配置错误。

## <a name="snowflake-connector-for-spark-notebooks"></a>Spark 笔记本的 Snowflake 连接器

以下笔记本提供有关如何将数据写入到 Snowflake 或从 Snowflake 读取数据的简单示例。 有关详细信息，请参阅[使用 Spark 连接器](https://docs.snowflake.com/en/user-guide/spark-connector-use.html)。 具体而言，请参阅[设置连接器的配置选项](https://docs.snowflake.com/en/user-guide/spark-connector-use.html#setting-configuration-options-for-the-connector)获取所有配置选项。

> [!TIP]
>
> 使用笔记本中演示的[机密](../../security/secrets/index.md#secrets-user-guide)，避免在笔记本中公开 Snowflake 用户名和密码。

### <a name="in-this-section"></a>本节内容：

* [Snowflake Scala 笔记本](#snowflake-scala-notebook)
* [Snowflake Python 笔记本](#snowflake-python-notebook)
* [Snowflake R 笔记本](#snowflake-r-notebook)

### <a name="snowflake-scala-notebook"></a>Snowflake Scala 笔记本

[获取笔记本](../../_static/notebooks/snowflake-scala.html)

### <a name="snowflake-python-notebook"></a>Snowflake Python 笔记本

[获取笔记本](../../_static/notebooks/snowflake-python.html)

### <a name="snowflake-r-notebook"></a>Snowflake R 笔记本

[获取笔记本](../../_static/notebooks/snowflake-r.html)

## <a name="train-a-machine-learning-model-and-save-results-to-snowflake"></a>训练机器学习模型，并将结果保存到 Snowflake

以下笔记本介绍如何使用适用于 Spark 的 Snowflake 连接器的最佳做法。 它将数据写入到 Snowflake，使用 Snowflake 进行一些基本的数据操作，训练 Azure Databricks 中的机器学习模型，并将结果写回 Snowflake。

### <a name="store-ml-training-results-in-snowflake-notebook"></a>在 Snowflake 笔记本中存储 ML 训练结果

[获取笔记本](../../_static/notebooks/snowflake-ml.html)

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

为什么我的 Spark 数据帧列在 Snowflake 中的顺序不相同？

适用于 Spark 的 Snowflake 连接器不遵循要写入的表中的列的顺序；必须显式指定数据帧和 Snowflake 列之间的映射。 若要指定此映射，请使用 [columnmap 参数](https://docs.snowflake.net/manuals/user-guide/spark-connector-use.html#setting-configuration-options-for-the-connector)。

为什么向 Snowflake 写入的 `INTEGER` 数据总是作为 `DECIMAL` 读回？

Snowflak 将所有 `INTEGER` 类型都表示为 `NUMBER`，这可能会导致在将数据写入到 Snowflak 并从中读取数据时数据类型发生更改。  例如，在写入 Snowflak 时可以将 `INTEGER` 数据转换为 `DECIMAL`，因为 `INTEGER` 和 `DECIMAL` 在 Snowflak 中是等效的（请参阅 [Snowflak 数字数据类型](https://docs.snowflake.net/manuals/sql-reference/data-types-numeric.html#int-integer-bigint-smallint-tinyint-byteint)）。

为什么 Snowflak 表架构中的字段总是大写的？

默认情况下，Snowflak 使用大写字段，这意味着表架构将转换为大写字母。