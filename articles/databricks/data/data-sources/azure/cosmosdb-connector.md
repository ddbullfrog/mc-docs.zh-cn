---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: Azure Cosmos DB - Azure Databricks
description: 了解如何使用 Azure Databricks 读取数据并将数据写入 Azure Cosmos DB。
ms.openlocfilehash: 4acce42d986bb2048f110946de1a01ce6a81216a
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121883"
---
# <a name="azure-cosmos-db"></a>Azure Cosmos DB

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

[Azure Cosmos DB](/cosmos-db/) 是由 Microsoft 提供的全球分布式多模型数据库。 使用 Azure Cosmos DB 可跨任意数量的 Azure 地理区域弹性且独立地缩放吞吐量和存储。
它通过综合服务级别协议 (SLA) 提供吞吐量、延迟、可用性和一致性保证。 Azure Cosmos DB 为以下数据模型提供 API，并提供多种语言的 SDK：

* SQL API
* MongoDB API
* Cassandra API
* 图形 (Gremlin) API
* 表 API

本文介绍如何从 Azure Cosmos DB 读取数据或将数据写入 Azure Cosmos DB。

> [!NOTE]
>
> 无法从运行 Databricks Runtime 7.0 或更高版本的群集访问此数据源，因为支持 Apache Spark 3.0 的 Azure Cosmos DB 连接器不可用。

## <a name="create-and-attach-required-libraries"></a>创建并附加所需的库

1. 下载最新版 azure-cosmosdb-spark 库以获取你正在运行的 Apache Spark 版本：
   * Spark 2.4: [azure-cosmosdb-spark_2.4.0_2.11-2.1.2-uber.jar](https://search.maven.org/remotecontent?filepath=com/microsoft/azure/azure-cosmosdb-spark_2.4.0_2.11/2.1.2/azure-cosmosdb-spark_2.4.0_2.11-2.1.2-uber.jar)
   * Spark 2.3: [azure-cosmosdb-spark_2.3.0_2.11-1.2.2-uber.jar](https://search.maven.org/remotecontent?filepath=com/microsoft/azure/azure-cosmosdb-spark_2.3.0_2.11/1.2.2/azure-cosmosdb-spark_2.3.0_2.11-1.2.2-uber.jar)
   * Spark 2.2: [azure-cosmosdb-spark_2.2.0_2.11-1.1.1-uber.jar](https://search.maven.org/remotecontent?filepath=com/microsoft/azure/azure-cosmosdb-spark_2.2.0_2.11/1.1.1/azure-cosmosdb-spark_2.2.0_2.11-1.1.1-uber.jar)
2. 按照[上传 Jar、Python Egg 或 Python Wheel](../../../libraries/workspace-libraries.md#uploading-libraries) 中的说明，将下载的 JAR 文件上传到 Databricks。
3. [安装上传的库](../../../libraries/cluster-libraries.md#install-libraries)，将其安装到 Databricks 群集中。

## <a name="use-the-azure-cosmos-db-spark-connector"></a>使用 Azure Cosmos DB Spark 连接器

下面的 Scala 笔记本提供了一个简单的示例，说明如何将数据写入 Cosmos DB 以及如何从 Cosmos DB 读取数据。 有关详细文档，请参阅 [Azure Cosmos DB Spark 连接器](https://github.com/Azure/azure-cosmosdb-spark)项目。 由 Microsoft 开发的 [Azure Cosmos DB Spark 连接器用户指南](https://github.com/Azure/azure-cosmosdb-spark/wiki/Azure-Cosmos-DB-Spark-Connector-User-Guide)也介绍了如何在 Python 中使用此连接器。

### <a name="azure-cosmos-db-notebook"></a>Azure Cosmos DB 笔记本

[获取笔记本](../../../_static/notebooks/cosmosdb.html)