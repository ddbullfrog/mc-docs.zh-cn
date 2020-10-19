---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: Delta Lake - Azure Databricks
description: 了解 Delta Lake 存储层以及随 Azure Databricks 上的 Delta Lake 提供的优化。
ms.openlocfilehash: 3a16055d389128b502c0254f705715d2b7483524
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121835"
---
# <a name="delta-lake"></a>Delta Lake

[Delta Lake](https://delta.io) 是可以提高 Data Lake 可靠性的[开源存储层](https://github.com/delta-io/delta)。 Delta Lake 提供 ACID 事务和可缩放的元数据处理，并可以统一流处理和批数据处理。 Delta Lake 在现有 Data Lake 的顶层运行，与 Apache Spark API 完全兼容。

利用 Azure Databricks 上的 Delta Lake，便可以根据工作负载模式配置 Delta Lake。 Azure Databricks 还包括 [Delta Engine](optimizations/index.md)，这为快速交互式查询提供了优化的布局和索引。

本部分介绍 Azure Databricks 上的 Delta Lake。

* [简介](delta-intro.md)
  * [快速入门](delta-intro.md#quickstart)
  * [资源](delta-intro.md#resources)
* [Delta Lake 快速入门](quick-start.md)
  * [创建表](quick-start.md#create-a-table)
  * [修改表](quick-start.md#modify-a-table)
  * [读取表](quick-start.md#read-a-table)
  * [优化表](quick-start.md#optimize-a-table)
  * [清理快照](quick-start.md#clean-up-snapshots)
* [介绍性笔记本](intro-notebooks.md)
  * [Delta Lake 快速入门 Python 笔记本](intro-notebooks.md#delta-lake-quickstart-python-notebook)
  * [Delta Lake 快速入门 Scala 笔记本](intro-notebooks.md#delta-lake-quickstart-scala-notebook)
  * [Delta Lake 快速入门 SQL 笔记本](intro-notebooks.md#delta-lake-quickstart-sql-notebook)
* [将数据引入到 Delta Lake](delta-ingest.md)
  * [合作伙伴集成](delta-ingest.md#partner-integrations)
  * [`COPY INTO` SQL 命令](delta-ingest.md#copy-into-sql-command)
  * [自动加载程序](delta-ingest.md#auto-loader)
* [表批量读取和写入](delta-batch.md)
  * [创建表](delta-batch.md#create-a-table)
  * [读取表](delta-batch.md#read-a-table)
  * [写入表](delta-batch.md#write-to-a-table)
  * [架构验证](delta-batch.md#schema-validation)
  * [更新表架构](delta-batch.md#update-table-schema)
  * [替换表架构](delta-batch.md#replace-table-schema)
  * [表中的视图](delta-batch.md#views-on-tables)
  * [表属性](delta-batch.md#table-properties)
  * [表元数据](delta-batch.md#table-metadata)
  * [笔记本](delta-batch.md#notebook)
* [表流读取和写入](delta-streaming.md)
  * [用作流源的增量表](delta-streaming.md#delta-table-as-a-stream-source)
  * [用作接收器的增量表](delta-streaming.md#delta-table-as-a-sink)
* [表删除、更新和合并](delta-update.md)
  * [从表中删除](delta-update.md#delete-from-a-table)
  * [更新表](delta-update.md#update-a-table)
  * [使用合并操作在表中执行更新插入](delta-update.md#upsert-into-a-table-using-merge)
  * [合并操作示例](delta-update.md#merge-examples)
* [表实用工具命令](delta-utility.md)
  * [清空](delta-utility.md#vacuum)
  * [描述历史记录](delta-utility.md#describe-history)
  * [描述详细信息](delta-utility.md#describe-detail)
  * [Convert to Delta](delta-utility.md#convert-to-delta)
  * [将 Delta 表转换为 Parquet 表](delta-utility.md#convert-a-delta-table-to-a-parquet-table)
  * [克隆 Delta 表](delta-utility.md#clone-a-delta-table)
* [表版本控制](versioning.md)
* [API 参考](delta-apidoc.md)
* [并发控制](concurrency-control.md)
  * [乐观并发控制](concurrency-control.md#optimistic-concurrency-control)
  * [写冲突](concurrency-control.md#write-conflicts)
  * [使用分区和非连续命令条件来避免冲突](concurrency-control.md#avoid-conflicts-using-partitioning-and-disjoint-command-conditions)
  * [冲突异常](concurrency-control.md#conflict-exceptions)
* [迁移指南](porting.md)
  * [将工作负荷迁移到 Delta Lake](porting.md#migrate-workloads-to-delta-lake)
* [最佳做法](best-practices.md)
  * [提供数据位置提示](best-practices.md#provide-data-location-hints)
  * [选择正确的分区列](best-practices.md#choose-the-right-partition-column)
  * [压缩文件](best-practices.md#compact-files)
  * [替换表的内容或架构](best-practices.md#replace-the-content-or-schema-of-a-table)
* [常见问题解答 (FAQ)](delta-faq.md)
  * [什么是 Delta Lake？](delta-faq.md#what-is-delta-lake)
  * [Delta Lake 与 Apache Spark 之间存在何种关系？](delta-faq.md#how-is-delta-lake-related-to-apache-spark)
  * [Delta Lake 使用哪种格式存储数据？](delta-faq.md#what-format-does-delta-lake-use-to-store-data)
  * [如何使用 Delta Lake 读取和写入数据？](delta-faq.md#how-can-i-read-and-write-data-with-delta-lake)
  * [Delta Lake 将数据存储在何处？](delta-faq.md#where-does-delta-lake-store-the-data)
  * [是否可将数据直接流式传入和流式传出 Delta 表？](delta-faq.md#can-i-stream-data-directly-into-and-from-delta-tables)
  * [Delta Lake 是否支持使用 Spark Streaming DStream API 写入或读取数据？](delta-faq.md#does-delta-lake-support-writes-or-reads-using-the-spark-streaming-dstream-api)
  * [使用 Delta Lake 时，是否可以轻松将代码移植到其他 Spark 平台？](delta-faq.md#when-i-use-delta-lake-will-i-be-able-to-port-my-code-to-other-spark-platforms-easily)
  * [增量表与 Hive SerDe 表之间有何差别？](delta-faq.md#how-do-delta-tables-compare-to-hive-serde-tables)
  * [Delta Lake 不支持哪些 DDL 和 DML 功能？](delta-faq.md#what-ddl-and-dml-features-does-delta-lake-not-support)
  * [Delta Lake 是否支持多表事务？](delta-faq.md#does-delta-lake-support-multi-table-transactions)
  * [如何更改列的类型？](delta-faq.md#how-can-i-change-the-type-of-a-column)
  * [Delta Lake 支持多群集写入是什么意思？](delta-faq.md#what-does-it-mean-that-delta-lake-supports-multi-cluster-writes)
  * [是否可从不同的工作区修改增量表？](delta-faq.md#can-i-modify-a-delta-table-from-different-workspaces)
  * [是否可以在 Databricks Runtime 的外部访问增量表？](delta-faq.md#can-i-access-delta-tables-outside-of-databricks-runtime)
* [资源](delta-resources.md)
  * [博客文章](delta-resources.md#blog-posts)
  * [讲座](delta-resources.md#talks)
  * [示例](delta-resources.md#examples)
  * [Delta Lake 事务日志规范](delta-resources.md#delta-lake-transaction-log-specification)