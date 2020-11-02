---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: Apache Kafka - Azure Databricks
description: 了解如何使用 Apache Kakfa 作为 Azure Databricks 中流式传输数据的源和接收器。
ms.openlocfilehash: b046548116633fce6f4fbdc6480349e2b5d2d2a1
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473105"
---
# <a name="apache-kafka"></a>Apache Kafka

用于结构化流式处理的 Apache Kafka 连接器在 Databricks Runtime 中打包。
可以使用 `kafka` 连接器连接到 Kafka 0.10+，使用 `kafka08` 连接器连接到 Kafka 0.8+（已弃用）。

## <a name="connect-kafka-on-hdinsight-to-azure-databricks"></a>将 Kafka on HDInsight 连接到 Azure Databricks

1. 创建 HDInsight Kafka 群集。

   有关说明，请参阅[通过 Azure 虚拟网络连接到 Kafka on HDInsight](/hdinsight/kafka/apache-kafka-connect-vpn-gateway)。

2. 对 Kafka 中转站进行配置以播发正确的地址。

   按照[为 IP 播发配置 Kafka](/hdinsight/kafka/apache-kafka-connect-vpn-gateway#configure-kafka-for-ip-advertising) 中的说明进行操作。 如果在 Azure 虚拟机上自行管理 Kafka，请确保将中转站的 `advertised.listeners` 配置设置为主机的内部 IP。

3. 创建 Azure Databricks 群集。

   遵循[快速入门：使用 Azure 门户在 Azure Databricks 上运行 Spark 作业](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal)。

4. 将 Kafka 群集与 Azure Databricks 群集进行对等互连。

   按照[将虚拟网络对等互连](../../../administration-guide/cloud-configurations/azure/vnet-peering.md)中的说明进行操作。

5. 通过测试[快速入门](#quickstart)和[使用 Kafka 笔记本的生产结构化流式处理](#production-kafka)中所述的方案来验证连接。

## <a name="schema"></a>架构

记录的架构为：

| 列            | 类型       |
|-------------------|------------|
| key               | binary     |
| value             | binary     |
| 主题             | string     |
| partition         | int        |
| offset            | long       |
| timestamp         | long       |
| timestampType     | int        |

始终使用 `ByteArrayDeserializer` 将 `key` 和 `value` 反序列化为字节数组。
请使用数据帧操作（`cast("string")`、udfs）显式反序列化键和值。

## <a name="quickstart"></a>快速入门

让我们从一个规范的 WordCount 示例开始。 以下笔记本演示了如何通过 Kafka 使用结构化流式处理来运行 WordCount。

> [!NOTE]
>
> 此笔记本示例使用 Kafka 0.10。 若要使用 Kafka 0.8，请将格式更改为 `kafka08`（即 `.format("kafka08")`）。

### <a name="kafka-wordcount-with-structured-streaming-notebook"></a>采用结构化流式处理笔记本的 Kafka WordCount

[获取笔记本](../../../_static/notebooks/structured-streaming-kafka.html)

## <a name="configuration"></a>配置

有关完整的配置列表，请参阅 [Spark 结构化流式处理 + Kafka 集成指南](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)。 若要开始，请参阅下面的部分配置。

> [!NOTE]
>
> 由于结构化流式处理仍处于开发阶段，所以此列表可能不是最新的。

有多种方法可以指定要订阅的主题。 你应当只提供以下参数之一：

| 选项               | 值                                               | 支持的 Kafka 版本     | 说明                                   |
|----------------------|-----------------------------------------------------|------------------------------|-----------------------------------------------|
| subscribe            | 以逗号分隔的主题列表。                   | 0.8、0.10                    | 要订阅的主题列表。               |
| subscribePattern     | Java 正则表达式字符串。                                  | 0.10                         | 用来订阅主题的模式。    |
| assign               | JSON 字符串 `{"topicA":[0,1],"topic":[2,4]}`。       | 0.8、0.10                    | 要使用的特定 topicPartition。          |

其他值得注意的配置：

| 选项                      | 值                                 | 默认值     | 支持的 Kafka 版本     | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|-----------------------------|---------------------------------------|-------------------|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| kafka.bootstrap.servers     | 以逗号分隔的 host:port 列表。    | empty             | 0.8、0.10                    | [必需] Kafka `bootstrap.servers` 配置。 如果没有发现来自 Kafka 的数据，请先检查中转站地址列表。 如果中转站地址列表不正确，则可能没有任何错误。 这是因为，Kafka 客户端假设中转站最终将变得可用，并且在出现网络错误的情况下会永远重试。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| failOnDataLoss              | `true` 或 `false`。                    | `true`            | 0.10                         | [可选] 在数据可能已丢失的情况下是否使查询失败。 在许多情况下（例如主题被删除、主题在处理前被截断，等等），查询可能永远也无法从 Kafka 读取数据。 我们会尝试保守地估计数据是否可能已丢失。 有时这可能会导致误报。 如果此选项无法正常工作，或者你希望查询不管数据是否丢失都继续进行处理，请将此选项设置为 `false`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| minPartitions               | >= 0 的整数，0 = 禁用。           | 0（禁用）      | 0.10                         | [可选] 要从 Kafka 读取的最小分区数。 在 Spark 2.1.0-db2 及更高版本中，可以使用 `minPartitions` 选项将 Spark 配置为使用任意最小数量分区从 Kafka 进行读取。 通常，Spark 在 Kafka topicPartitions 与从 Kafka 使用的 Spark 分区之间存在 1-1 映射。 如果将 `minPartitions` 选项设置为大于 Kafka topicPartitions 的值，则 Spark 会将大型 Kafka 分区拆分为较小的部分。 可以在出现峰值负载时、数据倾斜时以及在流落后时设置此选项以提高处理速度。 这需要在每次触发时初始化 Kafka 使用者。如果你在连接到 Kafka 时使用 SSL，这可能会影响性能。 此功能仅在 Databricks 中可用。                                                                                                                                                                                            |
| kafka.group.id              | Kafka 使用者组 ID。            | 未设置           | 0.10                         | [可选] 从 Kafka 进行读取时要使用的组 ID。 在 Spark 2.2+ 中受支持。 请谨慎使用此项。 默认情况下，每个查询都会生成用于读取数据的唯一组 ID。 这样可以确保每个查询都有其自己的使用者组，不会受到任何其他使用者的干扰，因此可以读取其订阅的主题的所有分区。 某些情况下（例如，在进行基于 Kafka 组的授权时），你可能希望使用特定的授权组 ID 来读取数据。 你还可以设置组 ID。 但是，执行此操作时要格外小心，因为它可能会导致意外的行为。<br><br>* 同时运行具有相同组 ID 的查询（包括批处理和流式处理）可能会相互干扰，导致每个查询仅读取部分数据。<br>* 快速连续启动/重启查询时，也可能会发生这种情况。   若要最大程度地减少此类问题，请将 Kafka 使用者配置 `session.timeout.ms` 设置为非常小的值。 |

有关其他可选配置，请参阅[结构化流式处理 Kafka 集成指南](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)。

> [!IMPORTANT]
>
> 不应为 Kafka 0.10 连接器设置以下 Kafka 参数，因为这将引发异常：
>
> * `group.id`：对于低于 2.2 的 Spark 版本，不允许设置此参数。
> * `auto.offset.reset`：请改为设置源选项 `startingOffsets` 来指定开始位置。 为了保持一致性，结构化流式处理（与 Kafka使用者相对）在内部管理偏移量的消耗。 这样可以确保你在动态订阅新主题/分区后不会错过任何数据。 `startingOffsets` 仅在你启动新的流式查询时应用，并且从检查点继续执行的操作总是从查询中断的地方开始。
> * `key.deserializer`：始终使用 `ByteArrayDeserializer` 将键反序列化为字节数组。 请使用数据帧操作来显式反序列化键。
> * `value.deserializer`：始终使用 `ByteArrayDeserializer` 将值反序列化为字节数组。 请使用数据帧操作来显式反序列化值。
> * `enable.auto.commit`：不允许设置此参数。 Spark 在内部跟踪 Kafka 偏移量，不提交任何偏移量。
> * `interceptor.classes`：Kafka 源始终将键和值作为字节数组进行读取。 使用 `ConsumerInterceptor` 不安全，因为它可能会破坏查询。

### <a name="production-structured-streaming-with-kafka-notebook"></a><a id="production-kafka"> </a><a id="production-structured-streaming-with-kafka-notebook"> </a>采用 Kafka 笔记本的生产结构化流式处理

[获取笔记本](../../../_static/notebooks/structured-streaming-etl-kafka.html)

## <a name="using-ssl"></a>使用 SSL

若要启用到 Kafka 的 SSL 连接，请按照 Confluent 文档[使用 SSL 进行加密和身份验证](https://docs.confluent.io/current/kafka/authentication_ssl.html#clients)中的说明进行操作。 你可以提供此处所述的配置（带前缀 `kafka.`）作为选项。 例如，在属性 `kafka.ssl.truststore.location` 中指定信任存储位置。

建议：

* 将证书存储在 Azure Blob 存储或 Azure Data Lake Storage Gen2 中，通过 [DBFS 装入点](../../../data/databricks-file-system.md#mount-storage)访问这些证书。 与[群集和作业 ACL](../../../administration-guide/access-control/index.md) 组合使用以后，就可以只允许能够访问 Kafka 的群集访问证书。
* 将证书密码存储为[机密作用域](../../../security/secrets/secret-scopes.md)中的[机密](../../../security/secrets/secrets.md)。

装载路径并存储机密后，你可以执行以下操作：

```python
df = spark.readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", ...) \
  .option("kafka.ssl.truststore.location", <dbfs-truststore-location>) \
  .option("kafka.ssl.keystore.location", <dbfs-keystore-location>) \
  .option("kafka.ssl.keystore.password", dbutils.secrets.get(scope=<certificate-scope-name>,key=<keystore-password-key-name>)) \
  .option("kafka.ssl.truststore.password", dbutils.secrets.get(scope=<certificate-scope-name>,key=<truststore-password-key-name>))
```

## <a name="resources"></a>资源

* [Apache Spark 结构化流式处理中与 Apache Kafka 的实时端到端集成](https://databricks.com/blog/2017/04/04/real-time-end-to-end-integration-with-apache-kafka-in-apache-sparks-structured-streaming.html)