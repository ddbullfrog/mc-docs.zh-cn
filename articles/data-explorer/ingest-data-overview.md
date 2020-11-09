---
title: Azure 数据资源管理器数据引入概述
description: 了解在 Azure 数据资源管理器中引入（加载）数据的不同方式
author: orspod
ms.author: v-tawe
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
origin.date: 05/18/2020
ms.date: 09/30/2020
ms.openlocfilehash: 81944720fe80978a700a2973e32b18c450ca7a83
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103618"
---
# <a name="azure-data-explorer-data-ingestion-overview"></a>Azure 数据资源管理器数据引入概述 

数据引入是用于从一个或多个源加载数据记录以在 Azure 数据资源管理器中将数据导入表的过程。 引入后，数据即可用于查询。

下图显示了用于在 Azure 数据资源管理器中运行的端到端流，以及不同的引入方法。

:::image type="content" source="media/data-ingestion-overview/data-management-and-ingestion-overview.png" alt-text="数据引入和管理的概述方案":::

Azure 数据资源管理器数据管理服务负责数据引入，并执行以下过程：

Azure 数据资源管理器从外部源拉取数据，并从挂起的 Azure 队列读取请求。 将数据分批或流式传输到数据管理器。 批数据流入相同的数据库和表中，以优化引入吞吐量。 Azure 数据资源管理器验证初始数据并在必要时转换数据格式。 进一步的数据操作包括匹配架构、组织、编制索引、编码和压缩数据。 数据根据设置的保留策略保留在存储中。 然后，数据管理器将数据引入提交到引擎，在那里数据可供查询。 

## <a name="supported-data-formats-properties-and-permissions"></a>支持的数据格式、属性和权限

* **[支持的数据格式](ingestion-supported-formats.md)** 

* **[引入属性](ingestion-properties.md)** ：影响数据引入方式的属性（例如，标记、映射、创建时间）。

* **权限** ：若要引入数据，需要 [数据库引入器级别权限](kusto/management/access-control/role-based-authorization.md)。 其他操作（例如查询）可能需要数据库管理员、数据库用户或表管理员权限。

## <a name="batching-vs-streaming-ingestion"></a>批量引入与流式引入

* 批量引入执行数据批处理，并针对高引入吞吐量进行优化。 此方法是引入的首选且性能最高。 数据根据引入属性进行批处理。 然后合并小批次的数据，并针对快速查询结果进行优化。 可以在数据库或表上设置[引入批处理](kusto/management/batchingpolicy.md)策略。 默认情况下，最大批处理值为 5 分钟、1000 项，或者 1 GB 的总大小。

* [流式引入](ingest-data-streaming.md)是来自流式处理源的正在进行的数据引入。 流式引入针对每个表的小型数据允许近乎实时的延迟。 数据最初引入到行存储，然后移动到列存储盘区。 可以使用 Azure 数据资源管理器客户端库或某个受支持的数据管道来完成流式引入。 

## <a name="ingestion-methods-and-tools"></a>引入方法和工具

Azure 数据资源管理器支持多种引入方法，每种方法都有自己的适用场景。 这些方法包括引入工具、各种服务的连接器和插件、托管管道、使用 SDK 的编程引入，以及直接引入。

### <a name="ingestion-using-managed-pipelines"></a>使用托管管道的引入

对于希望通过外部服务进行管理（限制、重试、监视器、警报等）的组织而言，使用连接器可能是最合适的解决方案。 排队引入适合大数据量。 Azure 数据资源管理器支持以下 Azure Pipelines：

<!-- * **[Event Grid](https://azure.microsoft.com/services/event-grid/)**: A pipeline that listens to Azure storage, and updates Azure Data Explorer to pull information when subscribed events occur. For more information, see [Ingest Azure Blobs into Azure Data Explorer](ingest-data-event-grid.md). -->

* **[事件中心](https://azure.microsoft.com/services/event-hubs/)** ：将事件从服务传输到 Azure 数据资源管理器的管道。 有关详细信息，请参阅[将数据从事件中心引入到 Azure 数据资源管理器](ingest-data-event-hub.md)。

* **[IoT 中心](https://azure.microsoft.com/services/iot-hub/)** ：用于将数据从支持的 IoT 设备传输到 Azure 数据资源管理器的管道。 有关详细信息，请参阅[从 IoT 中心引入](ingest-data-iot-hub.md)。

* **Azure 数据工厂 (ADF)** ：用于 Azure 中分析工作负荷的完全托管的数据集成服务。 Azure 数据工厂与 90 多个受支持的源连接，提供高效且可复原的数据传输。 ADF 准备、转换并丰富数据，以提供可通过不同方式进行监视的见解。 此服务可在周期性时间线上用作一次性解决方案，或者由特定的事件触发。 
  * [将 Azure 数据资源管理器与 Azure 数据工厂集成](data-factory-integration.md)。
  * [使用 Azure 数据工厂将数据从受支持的源复制到 Azure 数据资源管理器](./data-factory-load-data.md)。
  * [使用 Azure 数据工厂模板从数据库批量复制到 Azure 数据资源管理器](data-factory-template.md)。
  * [使用 Azure 数据工厂命令活动运行 Azure 数据资源管理器控制命令](data-factory-command-activity.md)。

### <a name="ingestion-using-connectors-and-plugins"></a>使用连接器和插件的引入

* Logstash 插件，请参阅[将数据从 Logstash 引入 Azure 数据资源管理器](ingest-data-logstash.md)。

* Kafka 连接器，请参阅[将数据从 Kafka 引入到 Azure 数据资源管理器](ingest-data-kafka.md)。

<!-- * **[Power Automate](https://flow.microsoft.com/)**: An automated workflow pipeline to Azure Data Explorer. Power Automate can be used to execute a query and do preset actions using the query results as a trigger. See [Azure Data Explorer connector to Power Automate (Preview)](flow.md). -->

* **Apache Spark 连接器** ：可在任何 Spark 群集上运行的开放源代码项目。 它实现了用于跨 Azure 数据资源管理器和 Spark 群集移动数据的数据源和数据接收器。 可以构建面向数据驱动型方案的可缩放快速应用程序。 请参阅[适用于 Apache Spark 的 Azure 数据资源管理器连接器](spark-connector.md)。

### <a name="programmatic-ingestion-using-sdks"></a>使用 SDK 的编程引入

Azure 数据资源管理器提供可用于查询和数据引入的 SDK。 通过在引入期间和之后尽量减少存储事务，编程引入得到优化，可降低引入成本 (COG)。

**可用的 SDK 和开放源代码项目**

* [Python SDK](kusto/api/python/kusto-python-client-library.md)

* [.NET SDK](kusto/api/netfx/about-the-sdk.md)

* [Java SDK](kusto/api/java/kusto-java-client-library.md)

* [Node SDK](kusto/api/node/kusto-node-client-library.md)

* [REST API](kusto/api/netfx/kusto-ingest-client-rest.md)

* [GO API](kusto/api/golang/kusto-golang-client-library.md)

### <a name="tools"></a>工具

<!-- * **[One click ingestion](ingest-data-one-click.md)**: Enables you to quickly ingest data by creating and adjusting tables from a wide range of source types. One click ingestion automatically suggests tables and mapping structures based on the data source in Azure Data Explorer. One click ingestion can be used for one-time ingestion, or to define continuous ingestion via Event Grid on the container to which the data was ingested. -->

* **[LightIngest](lightingest.md)** ：用于将即席数据引入 Azure 数据资源管理器的命令行实用工具。 该实用工具可以从本地文件夹或 Azure Blob 存储容器提取源数据。

### <a name="kusto-query-language-ingest-control-commands"></a>Kusto 查询语言引入控制命令

可以通过多种方法利用 Kusto 查询语言 (KQL) 命令将数据直接引入引擎。 由于此方法绕过数据管理服务，因此仅适用于探索和制作原型。 不要在生产或大容量方案中使用此方法。

  * **内联引入** ：向引擎发送控制命令 [.ingest inline](kusto/management/data-ingestion/ingest-inline.md)，要引入的数据是命令文本自身的一部分。 此方法用于临时测试目的。

  * **从查询引入** ：向引擎发送控制命令 [.set、.append、.set-or-append 或 .set-or-replace](kusto/management/data-ingestion/ingest-from-query.md)，将数据间接指定为查询或命令的结果。

  * **从存储引入（拉取）** ：向引擎发送控制命令 [.ingest into](kusto/management/data-ingestion/ingest-from-storage.md)，数据存储在某个外部存储（例如 Azure Blob 存储）中，可供引擎访问，命令也可以指向它。

## <a name="comparing-ingestion-methods-and-tools"></a>比较引入方法和工具

<!-- | [**One click ingestion**](ingest-data-one-click.md) | *sv, JSON | 1 GB uncompressed (see note)| Batching to container, local file and blob in direct ingestion | One-off, create table schema, definition of continuous ingestion with event grid, bulk ingestion with container (up to 10,000 blobs) | 10,000 blobs are randomly selected from container| -->
<!-- | [**Event Grid**](ingest-data-event-grid-overview.md) | [Supported data formats](ingest-data-event-grid-overview.md#data-format) | 1 GB uncompressed | Batching | Continuous ingestion from Azure storage, external data in Azure storage | 100 KB is optimal file size, Used for blob renaming and blob creation | -->
<!-- |[ **Azure Data Flow**](kusto/tools/flow.md) | | | | Ingestion commands as part of flow| Must have high-performing response time | -->

| 引入名称 | 数据类型 | 文件大小上限 | 流式引入，批量引入，直接引入 | 最常用场景 | 注意事项 |
| --- | --- | --- | --- | --- | --- |
| [**LightIngest**](lightingest.md) | 支持所有支持 | 1 GB，解压缩（参见备注） | 通过 DM 批量引入或直接引入引擎 |  数据迁移，含已调整引入时间戳的历史数据，批量引入（无大小限制）| 区分大小写，区分有无空格 |
| [**ADX Kafka**](ingest-data-kafka.md) | | | | |
| [**ADX 到 Apache Spark**](spark-connector.md) | | | | |
| [**LogStash**](ingest-data-logstash.md) | | | | |
<!-- | [**Azure 数据工厂**](kusto/tools/azure-data-factory.md) | [支持的数据格式](/data-factory/copy-activity-overview#supported-data-stores-and-formats) | 无限制 *（根据 ADF 限制） | 批量引入或者按 ADF 触发器引入 | 支持通常不受支持的格式及大型文件，可从 90 多个源进行复制，可从本地复制到云 | 引入时间 | -->
| [**IoT 中心**](ingest-data-iot-hub-overview.md) | [支持的数据格式](ingest-data-iot-hub-overview.md#data-format)  | 空值 | 批量引入，流式引入 | IoT 消息，IoT 事件，IoT 属性 | |
| [**事件中心**](ingest-data-event-hub-overview.md) | [支持的数据格式](ingest-data-event-hub-overview.md#data-format) | 空值 | 批量引入，流式引入 | 消息，事件 | |
| [**.NET SDK**](./net-sdk-ingest-data.md) | 支持所有格式 | 1 GB，解压缩（参见备注） | 批量引入，流式引入，直接引入 | 根据组织需求编写自己的代码 |
| [**Python**](python-ingest-data.md) | 支持所有格式 | 1 GB，解压缩（参见备注） | 批量引入，流式引入，直接引入 | 根据组织需求编写自己的代码 |
| [**Node.js**](node-ingest-data.md) | 支持所有格式 | 1 GB，解压缩（参见备注） | 批量引入，流式引入，直接引入 | 根据组织需求编写自己的代码 |
| [**Java**](kusto/api/java/kusto-java-client-library.md) | 支持所有格式 | 1 GB，解压缩（参见备注） | 批量引入，流式引入，直接引入 | 根据组织需求编写自己的代码 |
| [**REST**](kusto/api/netfx/kusto-ingest-client-rest.md) | 支持所有格式 | 1 GB，解压缩（参见备注） | 批量引入，流式引入，直接引入| 根据组织需求编写自己的代码 |
| [**Go**](kusto/api/golang/kusto-golang-client-library.md) | 支持所有格式 | 1 GB，解压缩（参见备注） | 批量引入，流式引入，直接引入 | 根据组织需求编写自己的代码 |

> [!Note] 
> 在上表中进行引用时，引入支持的最大文件大小为 4 GB。 建议引入 100 MB 到 1 GB 的文件。

## <a name="ingestion-process"></a>引入过程

根据需要选择最适合的引入方法后，执行以下步骤：

1. **设置保留策略**

    将数据引入 Azure 数据资源管理器中的表必须遵循该表的有效保留策略。 除非在表上显式设置，否则有效保留策略派生自数据库的保留策略。 热保留是群集大小和保留策略的功能。 引入超过可用空间的数据将强制首先进入的数据进行冷保留。
    
    确保数据库的保留策略符合你的需求。 如果并非如此，请在表级别显式重写它。 有关详细信息，请参阅[保留策略](kusto/management/retentionpolicy.md)。 
    
1. **创建表**

    为了引入数据，需要事先创建一个表。 使用以下选项之一：
   * [使用命令](kusto/management/create-table-command.md)创建表。 

   <!-- * Create a table using [One Click Ingestion](one-click-ingestion-new-table.md). -->

    > [!Note]
    > 如果记录不完整或者无法将字段解析为所需的数据类型，则将使用 NULL 值填充相应的表列。

1. **创建架构映射**

    [架构映射](kusto/management/mappings.md)有助于将源数据字段绑定到目标表列。 映射允许根据定义的属性，将不同源中的数据引入同一个表。 支持不同类型的映射，行导向（CSV、JSON 和 AVRO）和列导向 (Parquet)。 在大部分方法中，可以[在表上预先创建](kusto/management/create-ingestion-mapping-command.md)映射并从引入命令参数进行引用。

1. **设置更新策略** （可选）

   一些数据格式映射（Parquet、JSON 和 Avro）支持简单且有用的引入时间转换。 如果在引入时需要进行更复杂的处理，可使用更新策略，该策略允许使用 Kusto 查询语言命令进行轻型处理。 更新策略自动对原始表上的引入数据运行提取和转换，并将生成的数据引入到一个或多个目标表。 设置[更新策略](kusto/management/update-policy.md)。



## <a name="next-steps"></a>后续步骤

* [支持的数据格式](ingestion-supported-formats.md)
* [支持的引入属性](ingestion-properties.md)
