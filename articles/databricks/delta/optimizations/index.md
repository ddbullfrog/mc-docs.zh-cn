---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: Delta Engine - Azure Databricks
description: 了解可对 Delta Engine 使用的优化措施。
ms.openlocfilehash: bb7f72ce29863380a18d55a75557c7da08cd504c
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121940"
---
# <a name="delta-engine"></a>Delta Engine

Delta Engine 是与 Apache Spark 兼容的高性能查询引擎，提供了一种高效的方式来处理数据湖中的数据，包括存储在开源 Delta Lake 中的数据。 Delta Engine 优化可加快数据湖操作速度，并支持各种工作负载，从大规模 ETL 处理到临时交互式查询均可。 其中许多优化都自动进行；只需要通过将 Azure Databricks 用于数据湖即可获得这些 Delta Engine 功能的优势。

* [通过文件管理优化性能](file-mgmt.md)
  * [压缩（二进制打包）](file-mgmt.md#compaction-bin-packing)
  * [跳过数据](file-mgmt.md#data-skipping)
  * [Z 顺序（多维聚类）](file-mgmt.md#z-ordering-multi-dimensional-clustering)
  * [Notebook](file-mgmt.md#notebooks)
  * [提高交互式查询性能](file-mgmt.md#improve-interactive-query-performance)
  * [常见问题解答 (FAQ)](file-mgmt.md#frequently-asked-questions-faq)
* [自动优化](auto-optimize.md)
  * [自动优化的工作原理](auto-optimize.md#how-auto-optimize-works)
  * [使用情况](auto-optimize.md#usage)
  * [何时选择加入和选择退出](auto-optimize.md#when-to-opt-in-and-opt-out)
  * [示例工作流：使用并发删除或更新操作进行流式引入](auto-optimize.md#example-workflow-streaming-ingest-with-concurrent-deletes-or-updates)
  * [常见问题解答 (FAQ)](auto-optimize.md#frequently-asked-questions-faq)
* [通过缓存优化性能](delta-cache.md)
  * [增量缓存和 Apache Spark 缓存](delta-cache.md#delta-and-apache-spark-caching)
  * [增量缓存一致性](delta-cache.md#delta-cache-consistency)
  * [使用增量缓存](delta-cache.md#use-delta-caching)
  * [缓存一部分数据](delta-cache.md#cache-a-subset-of-the-data)
  * [监视增量缓存](delta-cache.md#monitor-the-delta-cache)
  * [配置增量缓存](delta-cache.md#configure-the-delta-cache)
* [动态文件修剪](dynamic-file-pruning.md)
* [隔离级别](isolation-level.md)
  * [设置隔离级别](isolation-level.md#set-the-isolation-level)
* [Bloom 筛选器索引](bloom-filters.md)
  * [配置](bloom-filters.md#configuration)
  * [创建 Bloom 筛选器索引](bloom-filters.md#create-a-bloom-filter-index)
  * [删除 Bloom 筛选器索引](bloom-filters.md#drop-a-bloom-filter-index)
  * [笔记本](bloom-filters.md#notebook)
* [优化联接性能](../join-performance/index.md)
  * [范围联接优化](../join-performance/range-join.md)
  * [倾斜联接优化](../join-performance/skew-join.md)
* [优化的数据转换](../data-transformation/index.md)
  * [高阶函数](../data-transformation/higher-order-lambda-functions.md)
  * [转换复杂数据类型](../data-transformation/complex-types.md)