---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/18/2020
title: Neo4j - Azure Databricks
description: 了解如何使用 Azure Databricks 在 Neo4j 中读取和写入数据。
ms.openlocfilehash: 9f44599f0f4e5e9fc8d8e0e9f4d0f56a3f967739
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121842"
---
# <a name="neo4j"></a>Neo4j

[Neo4j](https://neo4j.com/) 是一个利用数据关系作为第一类实体的本机图形数据库。 可以使用 [neo4j-spark-connector](https://github.com/neo4j-contrib/neo4j-spark-connector)（为 RDD、DataFrame、GraphX 和 GraphFrames 提供 Apache Spark API）将 Azure Databricks 群集连接到 Neo4j 群集。 neo4j-spark-connector 使用二进制 Bolt 协议将数据传输到 Neo4j 服务器以及从其传输数据。

本文介绍如何部署和配置 Neo4j、如何配置 Azure Databricks 以访问 Neo4j，并提供演示用法的笔记本。

> [!NOTE]
>
> 无法从运行 Databricks Runtime 7.0 或更高版本的群集访问此数据源，因为支持 Apache Spark 3.0 的 Neo4j 连接器不可用。

## <a name="neo4j-deployment-and-configuration"></a>Neo4j 部署和配置

可以在不同的云提供商上部署 Neo4j。

若要部署 Neo4j，请参阅官方的 Neo4j [云部署](https://neo4j.com/developer/guide-cloud-deployment/)指南。 本指南假定你使用 Neo4j 3.2.2。

更改 Neo4j 默认密码（首次访问 Neo4j 时系统会提示你），并修改 `conf/neo4j.conf` 以接受远程连接。

```ini
# conf/neo4j.conf

# Bolt connector
dbms.connector.bolt.enabled=true
#dbms.connector.bolt.tls_level=OPTIONAL
dbms.connector.bolt.listen_address=0.0.0.0:7687

# HTTP Connector. There must be exactly one HTTP connector.
dbms.connector.http.enabled=true
#dbms.connector.http.listen_address=0.0.0.0:7474

# HTTPS Connector. There can be zero or one HTTPS connectors.
dbms.connector.https.enabled=true
#dbms.connector.https.listen_address=0.0.0.0:7473
```

有关详细信息，请参阅[配置 Neo4j 连接器](https://neo4j.com/docs/operations-manual/current/configuration/connectors/)。

## <a name="azure-databricks-configuration"></a>Azure Databricks 配置

1. 安装两个库（[neo4j-spark-connector](https://spark-packages.org/package/neo4j-contrib/neo4j-spark-connector) 和 [graphframes](https://spark-packages.org/package/graphframes/graphframes)）作为 Spark 包。 有关说明，请参阅[库](../../libraries/workspace-libraries.md#maven-libraries)指南。
2. 使用这些 [Spark 配置](../../clusters/configure.md#spark-config)创建群集。

   ```bash
   spark.neo4j.bolt.url bolt://<ip-of-neo4j-instance>:7687
   spark.neo4j.bolt.user <username>
   spark.neo4j.bolt.password <password>
   ```

3. 导入库并测试连接。

   ```scala
   import org.neo4j.spark._
   import org.graphframes._

   val neo = Neo4j(sc)

   // Dummy Cypher query to check connection
   val testConnection = neo.cypher("MATCH (n) RETURN n;").loadRdd[Long]
   ```

### <a name="neo4j-notebook"></a>Neo4j 笔记本

[获取笔记本](../../_static/notebooks/neo4j.html)