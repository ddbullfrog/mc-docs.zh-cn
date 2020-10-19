---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/18/2020
title: 数据源 - Azure Databricks
description: 本文列出了与 Azure Databricks 兼容的 Apache Spark 数据源。
ms.openlocfilehash: 5b448b6d203b41562faf80b2a4be2985b78d93eb
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121814"
---
# <a name="data-sources"></a>数据源

本部分介绍可在 Azure Databricks 中使用的 Apache Spark 数据源。  很多都有一个笔记本来演示如何使用数据源读取和写入数据。

下列数据源直接在 Databricks Runtime 中受到支持，或者需要简单的 shell 命令来启用访问权限：

* [二进制文件](binary-file.md)
* [图像](image.md)
* [Hive 表](hive-tables.md)
* [MLflow 试验](mlflow-experiment.md)
* [Avro 文件](read-avro.md)
* [CSV 文件](read-csv.md)
* [JSON 文件](read-json.md)
* [LZO 压缩文件](read-lzo.md)
* [Parquet 文件](read-parquet.md)
* [Zip 文件](zip-files.md)

有关 Apache Spark 数据源的详细信息，请参阅[通用加载/保存函数](https://spark.apache.org/docs/latest/sql-data-sources-load-save-functions.html)和[泛型文件源选项](https://spark.apache.org/docs/latest/sql-data-sources-generic-options.html)。

以下存储数据源要求你配置与存储的连接。 一些还需要你创建一个 Azure Databricks [库](../../libraries/index.md)并将它安装在群集中：

* [Azure Blob 存储](azure/azure-storage.md)
* [Azure Data Lake Storage Gen2](azure/azure-datalake-gen2.md)
* [Azure Cosmos DB](azure/cosmosdb-connector.md)
* [Azure Synapse Analytics](azure/synapse-analytics.md)
* [使用 JDBC 的 SQL 数据库](sql-databases.md)
* [使用 Apache Spark 连接器的 SQL 数据库](sql-databases-azure.md)
* [Cassandra](cassandra.md)
* [Couchbase](couchbase.md)
* [ElasticSearch](elasticsearch.md)
* [MongoDB](mongodb.md)
* [Neo4j](neo4j.md)
* [Redis](redis.md)
* [Riak 时序](riak-ts.md)
* [Snowflake](snowflake.md)