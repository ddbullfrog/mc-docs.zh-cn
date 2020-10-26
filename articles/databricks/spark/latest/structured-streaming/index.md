---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 结构化流 - Azure Databricks
description: 了解如何使用 Apache Spark 结构化流基于 Azure Databricks 中的流数据表达计算。
ms.openlocfilehash: 96ba470a26e9006914f15c6539570ff9e6d07896
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472980"
---
# <a name="structured-streaming"></a>结构化流式处理

结构化流是一个 Apache Spark API，可让你基于流数据表达计算，就像基于静态数据表达批处理计算一样。 Spark SQL 引擎以增量方式执行计算，并在流数据抵达时持续更新结果。 有关结构化流的概述，请参阅 Apache Spark [结构化流编程指南](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)。
以下文章提供了介绍性笔记本、有关如何使用特定类型的流源和接收器、如何将流投入生产的详细信息，以及用于演示示例用例的笔记本：

* [演示笔记本](demo-notebooks.md)
  * [结构化流演示 Python 笔记本](demo-notebooks.md#structured-streaming-demo-python-notebook)
  * [结构化流演示 Scala 笔记本](demo-notebooks.md#structured-streaming-demo-scala-notebook)
* [流式处理数据源和接收器](data-sources.md)
  * [Apache Kafka](kafka.md)
  * [使用自动加载程序从 Azure Blob 存储或 Azure Data Lake Storage Gen2 加载文件](auto-loader.md)
  * [Azure 事件中心](streaming-event-hubs.md)
  * [Delta Lake 表](delta.md)
  * [读取和写入流 Avro 数据](avro-dataframe.md)
  * [写入到任意数据接收器](foreach.md)
  * [将优化的 Azure Blob 存储文件源与 Azure 队列存储配合使用](aqs.md)
* [生产中的结构化流式处理](production.md)
  * [在发生查询失败后进行恢复](production.md#recover-from-query-failures)
  * [配置 Apache Spark 计划程序池以提高效率](production.md#configure-apache-spark-scheduler-pools-for-efficiency)
  * [优化有状态流查询的性能](production.md#optimize-performance-of-stateful-streaming-queries)
  * [多水印策略](production.md#multiple-watermark-policy)
  * [触发器](production.md#triggers)
  * [可视化效果](production.md#visualizations)
* [结构化流示例](examples.md)
  * [使用 Scala 中的 `foreachBatch()` 写入到 Cassandra](examples.md#write-to-cassandra-using-foreachbatch-in-scala)
  * [使用 Python 中的 `foreachBatch()` 写入到 Azure Synapse Analytics](examples.md#write-to-azure-synapse-analytics-using-foreachbatch-in-python)
  * [流之间的联接](examples.md#stream-stream-joins)

有关结构化流的参考信息，Azure Databricks 建议参阅以下 Apache Spark API 参考文章：

* [Python](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#module-pyspark.sql.streaming)
* [Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.streaming.package)
* [Java](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/streaming/package-summary.html)

有关如何使用 Apache Spark 执行复杂流分析的详细信息，请参阅以下由多个部分组成的博客系列中的文章：

* [使用结构化流执行实时流 ETL](https://databricks.com/blog/2017/01/19/real-time-streaming-etl-structured-streaming-apache-spark-2-1.html)
* [使用结构化流处理复杂数据格式](https://databricks.com/blog/2017/02/23/working-complex-data-formats-structured-streaming-apache-spark-2-1.html)
* [使用结构化流处理 Apache Kafka 中的数据](https://databricks.com/blog/2017/04/26/processing-data-in-apache-kafka-with-structured-streaming-in-apache-spark-2-2.html)
* [Apache Spark 结构化流中的事件时间聚合和水印](https://databricks.com/blog/2017/05/08/event-time-aggregation-watermarking-apache-sparks-structured-streaming.html)
* [将 Apache Spark 的结构化流投入生产](https://databricks.com/blog/2017/05/18/taking-apache-sparks-structured-structured-streaming-to-production.html)
* [每天运行流式处理作业一次可实现 10 倍的成本节省：可缩放数据第 6 部分](https://databricks.com/blog/2017/05/22/running-streaming-jobs-day-10x-cost-savings.html)
* [Apache Spark 结构化流中的任意有状态处理](https://databricks.com/blog/2017/10/17/arbitrary-stateful-processing-in-apache-sparks-structured-streaming.html)

有关旧版 Spark 流功能的信息，请参阅：

* [Spark 流（旧版）](../rdd-streaming/index.md)