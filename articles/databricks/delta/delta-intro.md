---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/31/2020
title: 简介 - Azure Databricks
description: 了解 Delta Lake 的功能和资源以了解 Delta Lake。
ms.openlocfilehash: db85bba91f6fd78ee041e869db9fb83fcfaa7c36
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121889"
---
# <a name="introduction"></a>简介

[Delta Lake](https://delta.io) 是可提高[数据湖](https://databricks.com/discover/data-lakes/introduction)可靠性的[开源存储层](https://github.com/delta-io/delta)。 Delta Lake 提供 ACID 事务和可缩放的元数据处理，并可以统一流处理和批数据处理。  Delta Lake 在现有 Data Lake 的顶层运行，与 Apache Spark API 完全兼容。

具体而言，Delta Lake 提供：

* Spark 上的 ACID 事务：可序列化的隔离级别可避免读者看到不一致的数据。
* 可缩放的元数据处理：利用 Spark 的分布式处理能力，轻松处理包含数十亿文件的 PB 级表的所有元数据。
* 流式处理和批处理统一：Delta Lake 中的表是批处理表，也是流式处理源和接收器。 流式处理数据引入、批处理历史回填、交互式查询功能都是现成的。
* 架构强制：自动处理架构变体，以防在引入过程中插入错误的记录。
* 按时间顺序查看：数据版本控制支持回滚、完整的历史审核线索和可重现的机器学习试验。
* 更新插入和删除：支持合并、更新和删除操作，以启用复杂用例，如更改数据捕获、渐变维度 (SCD) 操作、流式处理更新插入等。

Delta Engine 优化使 Delta Lake 操作具有高性能，并支持各种工作负载，从大规模 ETL 处理到临时交互式查询均可。 有关 Delta Engine 的信息，请参阅 [Delta Engine](optimizations/index.md)。

## <a name="quickstart"></a>快速入门

Delta Lake 快速入门概述了使用 Delta Lake 的基础知识。 本[快速入门](quick-start.md)演示如何生成将 JSON 数据读取到 Delta 表中的管道以及如何修改表、读取表、显示表历史记录和优化表。

有关演示这些功能的 Azure Databricks 笔记本，请参阅[介绍性笔记本](intro-notebooks.md)。

若要试用 Delta Lake，请参阅[注册 Azure Databricks](/azure-databricks/quickstart-create-databricks-workspace-portal)。

## <a name="resources"></a>资源

* 有关常见问题的解答，请参阅[常见问题 (FAQ)](delta-faq.md)。
* 有关 Delta Lake SQL 命令的参考信息，请参阅[适用于 SQL 开发人员的 Azure Databricks](../spark/latest/spark-sql/index.md)。
* 有关更多资源（包括博客文章、讨论和示例），请参阅[资源](delta-resources.md)。