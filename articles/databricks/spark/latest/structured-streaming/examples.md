---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 结构化流示例 - Azure Databricks
description: 请参阅在 Azure Databricks 中通过 Cassandra、Azure Synapse Analytics、Python 笔记本和 Scala 笔记本使用 Spark 结构化流的示例。
ms.openlocfilehash: a273d7e8d4546bd07b78468111a9f03bbe59dfa5
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473123"
---
# <a name="structured-streaming-examples"></a>结构化流示例

* [Cassandra Scala 示例](#foreachbatch-cassandra-example)
* [Azure Synapse Analytics Python 示例](#foreachbatch-sqldw-example)
* [流间联接 Python 和 Scala 笔记本](#stream-stream-joins)

## <a name="write-to-cassandra-using-foreachbatch-in-scala"></a><a id="foreachbatch-cassandra-example"> </a><a id="write-to-cassandra-using-foreachbatch-in-scala"> </a>使用 Scala 中的 `foreachBatch()` 写入到 Cassandra

`streamingDF.writeStream.foreachBatch()` 允许你重复使用现有批数据写入器将流式处理查询的输出写入 Cassandra。 以下笔记本对此进行了演示，方法是：使用来自 Scala 的 Spark Cassandra 连接器将聚合查询的键/值输出写入 Cassandra。
有关详细信息，请参阅 [foreachBatch 文档](foreach.md)。

若要运行此示例，需要为 Spark 版本安装适当的 [Cassandra Spark 连接器](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector)，作为 [Maven 库](../../../libraries/workspace-libraries.md#maven-libraries)。

在此示例中，我们将创建一个表，然后启动结构化流式处理查询以将内容写入该表。
然后，我们使用 `foreachBatch()` 通过批数据帧连接器写入流输出。

```scala
import org.apache.spark.sql._
import org.apache.spark.sql.cassandra._

import com.datastax.spark.connector.cql.CassandraConnectorConf
import com.datastax.spark.connector.rdd.ReadConf
import com.datastax.spark.connector._

val host = "<ip address>"
val clusterName = "<cluster name>"
val keyspace = "<keyspace>"
val tableName = "<tableName>"

spark.setCassandraConf(clusterName, CassandraConnectorConf.ConnectionHostParam.option(host))
spark.readStream.format("rate").load()
  .selectExpr("value % 10 as key")
  .groupBy("key")
  .count()
  .toDF("key", "value")
  .writeStream
  .foreachBatch { (batchDF: DataFrame, batchId: Long) =>

    batchDF.write       // Use Cassandra batch data source to write streaming out
      .cassandraFormat(tableName, keyspace)
      .option("cluster", clusterName)
      .mode("append")
      .save()
  }
  .outputMode("update")
  .start()
```

## <a name="write-to-azure-synapse-analytics-using-foreachbatch-in-python"></a>使用 Python 中的 `foreachBatch()` 写入到 Azure Synapse Analytics

`streamingDF.writeStream.foreachBatch()` 允许你重复使用现有批数据写入器将流式处理查询的输出写入 Azure Synapse Analytics。 有关详细信息，请参阅 [foreachBatch 文档](foreach.md)。

若要运行此示例，需要 Azure Synapse Analytics 连接器。 有关 Azure Synapse Analytics 连接器的详细信息，请参阅 [Azure Synapse Analytics](../../../data/data-sources/azure/synapse-analytics.md)。

```python
from pyspark.sql.functions import *
from pyspark.sql import *

def writeToSQLWarehouse(df, epochId):
  df.write \
    .format("com.databricks.spark.sqldw") \
    .mode('overwrite') \
    .option("url", "jdbc:sqlserver://<the-rest-of-the-connection-string>") \
    .option("forward_spark_azure_storage_credentials", "true") \
    .option("dbtable", "my_table_in_dw_copy") \
    .option("tempdir", "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net/<your-directory-name>") \
    .save()

spark.conf.set("spark.sql.shuffle.partitions", "1")

query = (
  spark.readStream.format("rate").load()
    .selectExpr("value % 10 as key")
    .groupBy("key")
    .count()
    .toDF("key", "count")
    .writeStream
    .foreachBatch(writeToSQLWarehouse)
    .outputMode("update")
    .start()
    )
```

## <a name="stream-stream-joins"></a><a id="foreachbatch-sqldw-example"> </a><a id="stream-stream-joins"> </a>流间联接

这两个笔记本显示了如何在 Python 和 Scala 中使用流间联接。

### <a name="stream-stream-joins-python-notebook"></a>对 Python 笔记本进行流间联接

[获取笔记本](../../../_static/notebooks/stream-stream-joins-python.html)

### <a name="stream-stream-joins-scala-notebook"></a>对 Scala 笔记本进行流间联接

[获取笔记本](../../../_static/notebooks/stream-stream-joins-scala.html)