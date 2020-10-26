---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/05/2020
title: 资源 - Azure Databricks
description: 了解用于理解 Delta Lake 的资源。
ms.openlocfilehash: e73fdd10c3b03e1f7f8b0bfd96c3a35f03435946
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121901"
---
# <a name="resources"></a>资源

## <a name="blog-posts"></a>博客文章

* [Brand Safety with Spark Streaming and Delta Lake](https://www.eyeviewdigital.com/tech/brand-safety-with-spark-streaming-and-delta-lake/) - Eyeview
* [Simple, Reliable Upserts and Deletes on Delta Lake Tables using Python APIs](https://databricks.com/blog/2019/10/03/simple-reliable-upserts-and-deletes-on-delta-lake-tables-using-python-apis.html) - Databricks
* [Diving Into Delta Lake:Unpacking The Transaction Log](https://databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html) - Databricks
* [Diving Into Delta Lake:Schema Enforcement & Evolution](https://databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html) - Databricks
* [Introducing Delta Time Travel for Large Scale Data Lakes](https://databricks.com/blog/2019/02/04/introducing-delta-time-travel-for-large-scale-data-lakes.html) - Databricks
* [Productionizing Machine Learning with Delta Lake](https://databricks.com/blog/2019/08/14/productionizing-machine-learning-with-delta-lake.html) - Databricks
* [Simplifying Streaming Stock Analysis using Delta Lake and Apache Spark](https://databricks.com/blog/2019/06/18/simplifying-streaming-stock-analysis-using-delta-lake-and-apache-spark-on-demand-webinar-and-faq-now-available.html) - Databricks
* [Parallelizing SAIGE Across Hundreds of Cores](https://databricks.com/blog/2019/10/02/parallelizing-saige-across-hundreds-of-cores.html) - Databricks

请访问 [Databricks 博客](https://databricks.com/blog/category/delta-lake)以查看关于 Delta Lake 的最新帖子。

## <a name="talks"></a>谈话

* [Making Apache Spark Better with Delta Lake](https://vimeo.com/349735743) - Databricks
* [Delta Architecture, A Step Beyond Lambda Architecture](https://vimeo.com/352555281) - Databricks
* [Building Data Pipelines Using Structured Streaming and Delta Lake](https://databricks.com/session_eu19/designing-etl-pipelines-with-structured-streaming-and-delta-lake-how-to-architect-things-right) - Databricks
* [Building Reliable Data Lakes at Scale with Delta Lake](https://databricks.com/session_eu19/building-reliable-data-lakes-at-scale-with-delta-lake) - Databricks
  * 此自定进度的教程托管在 Delta Lake [Github 存储库](https://github.com/delta-io/delta/tree/master/examples/tutorials/saiseu19)上。
* [Near Real-Time Data Warehousing with Apache Spark and Delta Lake](https://databricks.com/session_eu19/near-real-time-data-warehousing-with-apache-spark-and-delta-lake) - Eventbrite
* [Power Your Delta Lake with Streaming Transactional Changes](https://databricks.com/session_eu19/power-your-delta-lake-with-streaming-transactional-changes) - StreamSets
* [Building an AI-Powered Retail Experience with Delta Lake, Spark, and Databricks](https://databricks.com/session_eu19/building-an-ai-powered-retail-experience-with-delta-lake-spark-and-databricks) - Zalando
* [Driver Location Intelligence at Scale using Apache Spark, Delta Lake, and MLflow on Databricks](https://databricks.com/session_eu19/driver-location-intelligence-at-scale-using-apache-spark-delta-lake-and-mlflow-on-databricks) - TomTom

## <a name="examples"></a>示例

Delta Lake Github 库具有 [Scala 和 Python 示例](https://github.com/delta-io/delta/tree/master/examples/)。

## <a name="delta-lake-transaction-log-specification"></a>Delta Lake 事务日志规范

Delta Lake 事务日志具有定义完善的开放协议，任何系统都可以使用该协议来读取日志。 请参阅 [Delta 事务日志协议](https://github.com/delta-io/delta/blob/master/PROTOCOL.md)。