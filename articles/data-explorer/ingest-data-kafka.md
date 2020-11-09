---
title: 将数据从 Kafka 引入到 Azure 数据资源管理器
description: 本文介绍如何将数据从 Kafka 引入（加载）到 Azure 数据资源管理器中。
author: orspod
ms.author: v-tawe
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: how-to
origin.date: 06/03/2019
ms.date: 09/30/2020
ms.openlocfilehash: 36b5206d565ab6776833a7ba9ea2132f8870c3ed
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104519"
---
# <a name="ingest-data-from-apache-kafka-into-azure-data-explorer"></a>将数据从 Apache Kafka 引入到 Azure 数据资源管理器中
 
Azure 数据资源管理器支持从 [Apache Kafka](http://kafka.apache.org/documentation/) 进行[数据引入](ingest-data-overview.md)。 Apache Kafka 是一个分布式流式处理平台，可用于构建实时流式处理数据管道，在系统或应用程序之间可靠地移动数据。 [Kafka Connect](https://docs.confluent.io/3.0.1/connect/intro.html#kafka-connect) 是一个工具，用于在 Apache Kafka 和其他数据系统之间以可缩放且可靠的方式流式传输数据。 Azure 数据资源管理器 [Kafka 接收器](https://github.com/Azure/kafka-sink-azure-kusto/blob/master/README.md)充当来自 Kafka 的连接器，并且不需要使用代码。 从此 [Git 存储库](https://github.com/Azure/kafka-sink-azure-kusto/releases)或 [Confluent 连接器中心](https://www.confluent.io/hub/microsoftcorporation/kafka-sink-azure-kusto)下载接收器连接器 jar。

本文介绍了如何通过 Kafka 将数据引入到 Azure 数据资源管理器，使用自包含 Docker 安装程序简化 Kafka 群集和 Kafka 连接器群集设置。 

有关详细信息，请参阅连接器 [Git 存储库](https://github.com/Azure/kafka-sink-azure-kusto/blob/master/README.md)和[版本具体信息](https://github.com/Azure/kafka-sink-azure-kusto/blob/master/README.md#13-major-version-specifics)。

## <a name="prerequisites"></a>先决条件

* 创建 [Azure 帐户](/)。
* 安装 [Azure CLI](/cli/install-azure-cli)。
* 安装 [Docker](https://docs.docker.com/get-docker/) 和 [Docker Compose](https://docs.docker.com/compose/install)。
* 使用默认缓存和保留策略[在 Azure 门户中创建 Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md)。

## <a name="create-an-azure-active-directory-service-principal"></a>创建 Azure Active Directory 服务主体

Azure Active Directory 服务主体可以通过 [Azure 门户](/active-directory/develop/howto-create-service-principal-portal)或通过编程方式进行创建，如以下示例所示。

此服务主体将是连接器用于写入到 Azure 数据资源管理器表的标识。 稍后我们将授予此服务主体访问 Azure 数据资源管理器所需的权限。

1. 通过 Azure CLI 登录到你的 Azure 订阅。 然后在浏览器中进行身份验证。

   ```azurecli
   az cloud set -n AzureChinaCloud
   az login
   ```


1. 选择要用于运行实验室的订阅。 当你有多个订阅时，此步骤是必需的。

   ```azurecli
   az account set --subscription YOUR_SUBSCRIPTION_GUID
   ```

1. 创建服务主体。 在此示例中，服务主体名为 `kusto-kafka-spn`。

   ```azurecli
   az ad sp create-for-rbac -n "kusto-kafka-spn"
   ```

1. 你将得到一个 JSON 响应，如下所示。 复制 `appId`、`password` 和 `tenant`，因为你在后面的步骤中将需要它们。

    ```json
    {
      "appId": "fe7280c7-5705-4789-b17f-71a472340429",
      "displayName": "kusto-kafka-spn",
      "name": "http://kusto-kafka-spn",
      "password": "29c719dd-f2b3-46de-b71c-4004fb6116ee",
      "tenant": "42f988bf-86f1-42af-91ab-2d7cd011db42"
    }
    ```

## <a name="create-a-target-table-in-azure-data-explorer"></a>在 Azure 数据资源管理器中创建目标表

1. 登录到 [Azure 门户](https://portal.azure.cn)

1. 转到你的 Azure 数据资源管理器群集。

1. 使用以下命令创建一个名为 `Storms` 的表：

    ```kusto
    .create table Storms (StartTime: datetime, EndTime: datetime, EventId: int, State: string, EventType: string, Source: string)
    ```

    :::image type="content" source="media/ingest-data-kafka/create-table.png" alt-text="在 Azure 数据资源管理器门户中创建表":::
    
1. 使用以下命令为引入的数据创建对应的表映射 `Storms_CSV_Mapping`：
    
    ```kusto
    .create table Storms ingestion csv mapping 'Storms_CSV_Mapping' '[{"Name":"StartTime","datatype":"datetime","Ordinal":0}, {"Name":"EndTime","datatype":"datetime","Ordinal":1},{"Name":"EventId","datatype":"int","Ordinal":2},{"Name":"State","datatype":"string","Ordinal":3},{"Name":"EventType","datatype":"string","Ordinal":4},{"Name":"Source","datatype":"string","Ordinal":5}]'
    ```    

1. 针对可配置的引入延迟，在表上创建批量引入策略。

    > [!TIP]
    > [引入批处理策略](kusto/management/batchingpolicy.md)是一个性能优化器，包含三个参数。 满足第一个参数将触发到 Azure 数据资源管理器表的引入。

    ```kusto
    .alter table Storms policy ingestionbatching @'{"MaximumBatchingTimeSpan":"00:00:15", "MaximumNumberOfItems": 100, "MaximumRawDataSizeMB": 300}'
    ```

1. 使用[创建 Azure Active Directory 服务主体](#create-an-azure-active-directory-service-principal)中的服务主体来授予使用数据库的权限。

    ```kusto
    .add database YOUR_DATABASE_NAME admins  ('aadapp=YOUR_APP_ID;YOUR_TENANT_ID') 'AAD App'
    ```

## <a name="run-the-lab"></a>运行实验室

以下实验室旨在为你提供开始创建数据、设置 Kafka 连接器以及使用连接器将此数据流式传输到 Azure 数据资源管理器的体验。 然后，你可以查看引入的数据。

### <a name="clone-the-git-repo"></a>克隆 git 存储库

克隆实验室的 git [存储库](https://github.com/Azure/azure-kusto-labs)。

1. 在计算机上创建一个本地目录。

    ```
    mkdir ~/kafka-kusto-hol
    cd ~/kafka-kusto-hol
    ```

1. 克隆存储库。

    ```shell
    cd ~/kafka-kusto-hol
    git clone https://github.com/Azure/azure-kusto-labs
    cd azure-kusto-labs/kafka-integration/dockerized-quickstart
    ```

#### <a name="contents-of-the-cloned-repo"></a>克隆的存储库的内容

运行以下命令以列出克隆的存储库的内容：

```
cd ~/kafka-kusto-hol/azure-kusto-labs/kafka-integration/dockerized-quickstart
tree
```

此搜索的该结果是：

```
├── README.md
├── adx-query.png
├── adx-sink-config.json
├── connector
│   └── Dockerfile
├── docker-compose.yaml
└── storm-events-producer
    ├── Dockerfile
    ├── StormEvents.csv
    ├── go.mod
    ├── go.sum
    ├── kafka
    │   └── kafka.go
    └── main.go
 ```

### <a name="review-the-files-in-the-cloned-repo"></a>查看克隆的存储库中的文件

以下各部分介绍了上述文件树中的文件的重要部分。

#### <a name="adx-sink-configjson"></a>adx-sink-config.json

此文件包含 Kusto 接收器属性文件，你将在其中更新特定配置详细信息：
 
```json
{
    "name": "storm",
    "config": {
        "connector.class": "com.microsoft.azure.kusto.kafka.connect.sink.KustoSinkConnector",
        "flush.size.bytes": 10000,
        "flush.interval.ms": 10000,
        "tasks.max": 1,
        "topics": "storm-events",
        "kusto.tables.topics.mapping": "[{'topic': 'storm-events','db': '<enter database name>', 'table': 'Storms','format': 'csv', 'mapping':'Storms_CSV_Mapping'}]",
        "aad.auth.authority": "<enter tenant ID>",
        "aad.auth.appid": "<enter application ID>",
        "aad.auth.appkey": "<enter client secret>",
        "kusto.url": "https://ingest-<name of cluster>.<region>.kusto.chinacloudapi.cn",
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.storage.StringConverter"
    }
}
```

根据你的 Azure 数据资源管理器设置替换以下属性的值：`aad.auth.authority`、`aad.auth.appid`、`aad.auth.appkey`、`kusto.tables.topics.mapping`（数据库名称）和 `kusto.url`。

#### <a name="connector---dockerfile"></a>连接器 - Dockerfile

此文件包含用于为连接器实例生成 docker 映像的命令。  它包括 git 存储库版本目录中的连接器下载。

#### <a name="storm-events-producer-directory"></a>Storm-events-producer 目录

此目录包含一个用于读取本地“StormEvents.csv”文件并将数据发布到 Kafka 主题的 Go 程序。

#### <a name="docker-composeyaml"></a>docker-compose.yaml

```yaml
version: "2"
services:
  zookeeper:
    image: debezium/zookeeper:1.2
    ports:
      - 2181:2181
  kafka:
    image: debezium/kafka:1.2
    ports:
      - 9092:9092
    links:
      - zookeeper
    depends_on:
      - zookeeper
    environment:
      - ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092
  kusto-connect:
    build:
      context: ./connector
      args:
        KUSTO_KAFKA_SINK_VERSION: 1.0.1
    ports:
      - 8083:8083
    links:
      - kafka
    depends_on:
      - kafka
    environment:
      - BOOTSTRAP_SERVERS=kafka:9092
      - GROUP_ID=adx
      - CONFIG_STORAGE_TOPIC=my_connect_configs
      - OFFSET_STORAGE_TOPIC=my_connect_offsets
      - STATUS_STORAGE_TOPIC=my_connect_statuses
  events-producer:
    build:
      context: ./storm-events-producer
    links:
      - kafka
    depends_on:
      - kafka
    environment:
      - KAFKA_BOOTSTRAP_SERVER=kafka:9092
      - KAFKA_TOPIC=storm-events
      - SOURCE_FILE=StormEvents.csv
```

### <a name="start-the-containers"></a>启动容器

1. 在终端启动容器：
    
    ```shell
    docker-compose up
    ```

    生成者应用程序将开始向 `storm-events` 主题发送事件。 
    你应当会看到类似于以下日志的日志：

    ```shell
    ....
    events-producer_1  | sent message to partition 0 offset 0
    events-producer_1  | event  2007-01-01 00:00:00.0000000,2007-01-01 00:00:00.0000000,13208,NORTH CAROLINA,Thunderstorm Wind,Public
    events-producer_1  | 
    events-producer_1  | sent message to partition 0 offset 1
    events-producer_1  | event  2007-01-01 00:00:00.0000000,2007-01-01 05:00:00.0000000,23358,WISCONSIN,Winter Storm,COOP Observer
    ....
    ```
    
1. 若要检查日志，请在单独的终端中运行以下命令：

    ```shell
    docker-compose logs -f | grep kusto-connect
    ```
    
### <a name="start-the-connector"></a>启动连接器

使用 Kafka Connect REST 调用来启动连接器。

1. 在单独的终端中，使用以下命令启动接收器任务：

    ```shell
    curl -X POST -H "Content-Type: application/json" --data @adx-sink-config.json http://localhost:8083/connectors
    ```
    
1. 若要检查状态，请在单独的终端中运行以下命令：
    
    ```shell
    curl http://localhost:8083/connectors/storm/status
    ```

连接器将开始对到 Azure 数据资源管理器的引入进程排队。

> [!NOTE]
> 如果有日志连接器问题，请[创建问题](https://github.com/Azure/kafka-sink-azure-kusto/issues)。

## <a name="query-and-review-data"></a>查询和查看数据

### <a name="confirm-data-ingestion"></a>确认数据引入

1. 等待数据到达 `Storms` 表。 若要确认数据的传输，请检查行计数：
    
    ```kusto
    Storms | count
    ```

1. 确认引入进程中没有失败：

    ```kusto
    .show ingestion failures
    ```
    
    看到数据后，请尝试一些查询。 

### <a name="query-the-data"></a>查询数据

1. 若要查看所有记录，请运行以下[查询](write-queries.md)：
    
    ```kusto
    Storms
    ```

1. 使用 `where` 和 `project` 来筛选特定数据：
    
    ```kusto
    Storms
    | where EventType == 'Drought' and State == 'TEXAS'
    | project StartTime, EndTime, Source, EventId
    ```
    
1. 使用 [`summarize`](./write-queries.md#summarize) 运算符：

    ```kusto
    Storms
    | summarize event_count=count() by State
    | where event_count > 10
    | project State, event_count
    | render columnchart
    ```
    
    :::image type="content" source="media/ingest-data-kafka/kusto-query.png" alt-text="Azure 数据资源管理器中的 Kafka 查询柱形图结果":::

如需更多的查询示例和指南，请参阅[针对 Azure 数据资源管理器编写查询](write-queries.md)和 [Kusto 查询语言文档](./kusto/query/index.md)。

## <a name="reset"></a>重置

若要进行重置，请执行以下步骤：

1. 停止容器 (`docker-compose down -v`)
1. 删除 (`drop table Storms`)
1. 重新创建 `Storms` 表
1. 重新创建表映射
1. 重启容器 (`docker-compose up`)

## <a name="clean-up-resources"></a>清理资源

若要删除 Azure 数据资源管理器资源，请使用 [az cluster delete](/cli/kusto/cluster#az-kusto-cluster-delete) 或 [az Kusto database delete](/cli/kusto/database#az-kusto-database-delete)：

```azurecli
az kusto cluster delete -n <cluster name> -g <resource group name>
az kusto database delete -n <database name> --cluster-name <cluster name> -g <resource group name>
```

## <a name="next-steps"></a>后续步骤

* 详细了解[大数据体系结构](/azure/architecture/solution-ideas/articles/big-data-azure-data-explorer)。
* 了解[如何将 JSON 格式的示例数据引入 Azure 数据资源管理器](./ingest-json-formats.md?tabs=kusto-query-language)。
* 对于其他 Kafka 实验室：
   * [在分布式模式下从 Confluent Cloud Kafka 进行引入的动手实验室](https://github.com/Azure/azure-kusto-labs/blob/master/kafka-integration/confluent-cloud/README.md)
   * [在分布式模式下从 HDInsight Kafka 进行引入的动手实验室](https://github.com/Azure/azure-kusto-labs/tree/master/kafka-integration/distributed-mode/hdinsight-kafka)
   * [在分布式模式下从 AKS 上的 Confluent IaaS Kafka 进行引入的动手实验室](https://github.com/Azure/azure-kusto-labs/blob/master/kafka-integration/distributed-mode/confluent-kafka/README.md)