---
title: 将 JSON 格式的数据引入 Azure 数据资源管理器
description: 了解如何将 JSON 格式的数据引入 Azure 数据资源管理器。
author: orspod
ms.author: v-tawe
ms.reviewer: kerend
ms.service: data-explorer
ms.topic: how-to
origin.date: 05/19/2020
ms.date: 09/30/2020
ms.openlocfilehash: 0626f5e870d125ea87e33a0d0b4681dbe49ef60b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103615"
---
# <a name="ingest-json-formatted-sample-data-into-azure-data-explorer"></a>将 JSON 格式的示例数据引入 Azure 数据资源管理器

本文介绍如何将 JSON 格式的数据引入 Azure 数据资源管理器数据库。 首先你将引入简单的原始映射 JSON 示例，再引入多行 JSON，然后处理包含数组和字典的更复杂 JSON 架构。  这些示例详细演示了使用 Kusto 查询语言 (KQL)、C# 或 Python 引入 JSON 格式数据的过程。 Kusto 查询语言 `ingest` 控制命令是直接对引擎终结点执行的。 在生产方案中，引入是使用客户端库或数据连接对数据管理服务执行的。 有关使用这些客户端库引入数据的演练，请阅读[使用 Azure 数据资源管理器 Python 库引入数据](python-ingest-data.md)和[使用 Azure 数据资源管理器 .NET Standard SDK 引入数据](./net-sdk-ingest-data.md)。

## <a name="prerequisites"></a>先决条件

[测试群集和数据库](create-cluster-database-portal.md)

## <a name="the-json-format"></a>JSON 格式

Azure 数据资源管理器支持两种 JSON 文件格式：
* `json`：行分隔的 JSON。 输入数据中的每一行只包含一条 JSON 记录。
* `multijson`：多行 JSON。 分析器将忽略行分隔符，并读取从前一位置到有效 JSON 末尾的一条记录。

### <a name="ingest-and-map-json-formatted-data"></a>引入和映射 JSON 格式的数据

引入 JSON *格式* 的数据需要使用 [引入属性](ingestion-properties.md)指定格式。 引入 JSON 数据需要执行[映射](kusto/management/mappings.md)，以将 JSON 源条目映射到其目标列。 引入数据时，将 `IngestionMapping` 属性与其 `ingestionMappingReference`（用于预定义的映射）引入属性或其 `IngestionMappings` 属性结合使用。 本文将使用 `ingestionMappingReference` 引入属性，该属性是在用于引入的表中预定义的。 以下示例首先将 JSON 记录作为原始数据引入到包含单个列的表中。 接下来，使用映射将每个属性引入到其映射列中。 

### <a name="simple-json-example"></a>简单 JSON 示例

以下示例是采用平面结构的简单 JSON。 数据包含多个设备收集的温度和湿度信息。 每条记录已使用 ID 和时间戳进行标记。

```json
{
    "timestamp": "2019-05-02 15:23:50.0369439",
    "deviceId": "2945c8aa-f13e-4c48-4473-b81440bb5ca2",
    "messageId": "7f316225-839a-4593-92b5-1812949279b3",
    "temperature": 31.0301639051317,
    "humidity": 62.0791099602725
}
```

## <a name="ingest-raw-json-records"></a>引入原始 JSON 记录 

此示例将 JSON 记录作为原始数据引入到包含单个列的表中。 使用查询和更新策略的数据操作是在引入数据后执行的。

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

使用 Kusto 查询语言引入原始 JSON 格式的数据。

1. 登录到 [https://dataexplorer.azure.cn](https://dataexplorer.azure.cn)。

1. 选择“添加群集”。

1. 在“添加群集”对话框中，以 `https://<ClusterName>.<Region>.kusto.chinacloudapi.cn/` 格式输入群集 URL，然后选择“添加”。

1. 粘贴以下命令，然后选择“运行”以创建表。

    ```kusto
    .create table RawEvents (Event: dynamic)
    ```

    此查询将创建一个表，其中包含单个[动态](kusto/query/scalar-data-types/dynamic.md)数据类型的 `Event` 列。

1. 创建 JSON 映射。

    ```kusto
    .create table RawEvents ingestion json mapping 'RawEventMapping' '[{"column":"Event","Properties":{"path":"$"}}]'
    ```

    此命令创建一个映射，以将 JSON 根路径 `$` 映射到 `Event` 列。

1. 将数据引入 `RawEvents` 表中。

    ```kusto
    .ingest into table RawEvents h'https://kustosamplefiles.blob.core.chinacloudapi.cn/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=json, jsonMappingReference=RawEventMapping)
    ```

# <a name="c"></a>[C#](#tab/c-sharp)

使用 C# 引入原始 JSON 格式的数据。

1. 创建 `RawEvents` 表。

    ```csharp
    var kustoUri = "https://<ClusterName>.<Region>.kusto.chinacloudapi.cn:443/";
    var kustoConnectionStringBuilder =
        new KustoConnectionStringBuilder(ingestUri)
        {
            FederatedSecurity = true,
            InitialCatalog = database,
            UserID = user,
            Password = password,
            Authority = tenantId
        };
    var kustoClient = KustoClientFactory.CreateCslAdminProvider(kustoConnectionStringBuilder);

    var table = "RawEvents";
    var command =
        CslCommandGenerator.GenerateTableCreateCommand(
            table,
            new[]
            {
                Tuple.Create("Events", "System.Object"),
            });

    kustoClient.ExecuteControlCommand(command);
    ```

1. 创建 JSON 映射。
    
    ```csharp
    var tableMapping = "RawEventMapping";
    var command =
        CslCommandGenerator.GenerateTableMappingCreateCommand(
            Data.Ingestion.IngestionMappingKind.Json,
            tableName,
            tableMapping,
            new[] {
            new ColumnMapping {ColumnName = "Events", Properties = new Dictionary<string, string>() {
                {"path","$"} }
            } });

    kustoClient.ExecuteControlCommand(command);
    ```
    此命令创建一个映射，以将 JSON 根路径 `$` 映射到 `Event` 列。

1. 将数据引入 `RawEvents` 表中。

    ```csharp
    var ingestUri = "https://ingest-<ClusterName>.<Region>.kusto.chinacloudapi.cn:443/";
    var blobPath = "https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var ingestConnectionStringBuilder =
        new KustoConnectionStringBuilder(ingestUri)
        {
            FederatedSecurity = true,
            InitialCatalog = database,
            UserID = user,
            Password = password,
            Authority = tenantId
        };
    var ingestClient = KustoIngestFactory.CreateQueuedIngestClient(ingestConnectionStringBuilder);
    
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.json,
            IngestionMapping = new IngestionMapping()
            {
                IngestionMappingReference = tableMapping
            }
        };

    ingestClient.IngestFromStorageAsync(blobPath, properties);
    ```

> [!NOTE]
> 数据是根据[批处理策略](kusto/management/batchingpolicy.md)聚合的，因此会出现几分钟的延迟。

# <a name="python"></a>[Python](#tab/python)

使用 Python 引入原始 JSON 格式的数据。

1. 创建 `RawEvents` 表。

    ```python
    KUSTO_URI = "https://<ClusterName>.<Region>.kusto.chinacloudapi.cn:443/"
    KCSB_DATA = KustoConnectionStringBuilder.with_aad_device_authentication(KUSTO_URI, AAD_TENANT_ID)
    KUSTO_CLIENT = KustoClient(KCSB_DATA)
    TABLE = "RawEvents"

    CREATE_TABLE_COMMAND = ".create table " + TABLE + " (Events: dynamic)"
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_TABLE_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. 创建 JSON 映射。

    ```python
    MAPPING = "RawEventMapping"
    CREATE_MAPPING_COMMAND = ".create table " + TABLE + " ingestion json mapping '" + MAPPING + """' '[{"column":"Event","path":"$"}]'"""
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_MAPPING_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. 将数据引入 `RawEvents` 表中。

    ```python
    INGEST_URI = "https://ingest-<ClusterName>.<Region>.kusto.chinacloudapi.cn:443/"
    KCSB_INGEST = KustoConnectionStringBuilder.with_aad_device_authentication(INGEST_URI, AAD_TENANT_ID)
    INGESTION_CLIENT = KustoIngestClient(KCSB_INGEST)
    BLOB_PATH = 'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.json, mappingReference=MAPPING)
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

    > [!NOTE]
    > 数据是根据[批处理策略](kusto/management/batchingpolicy.md)聚合的，因此会出现几分钟的延迟。

---

## <a name="ingest-mapped-json-records"></a>引入映射的 JSON 记录

此示例引入 JSON 记录数据。 每个 JSON 属性映射到表中的单个列。 

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

1. 创建一个新表，该表采用类似于 JSON 输入数据的架构。 我们将对下面的所有示例和引入命令使用此表。 

    ```kusto
    .create table Events (Time: datetime, Device: string, MessageId: string, Temperature: double, Humidity: double)
    ```

1. 创建 JSON 映射。

    ```kusto
    .create table Events ingestion json mapping 'FlatEventMapping' '[{"column":"Time","Properties":{"path":"$.timestamp"}},{"column":"Device","Properties":{"path":"$.deviceId"}},{"column":"MessageId","Properties":{"path":"$.messageId"}},{"column":"Temperature","Properties":{"path":"$.temperature"}},{"column":"Humidity","Properties":{"path":"$.humidity"}}]'
    ```

    在此映射中，根据表架构的定义，`timestamp` 条目将作为 `datetime` 数据类型引入到 `Time` 列。

1. 将数据引入 `Events` 表中。

    ```kusto
    .ingest into table Events h'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=json, jsonMappingReference=FlatEventMapping)
    ```

    文件“simple.json”包含几条行分隔的 JSON 记录。 格式为 `json`，在引入命令中使用的映射是创建的 `FlatEventMapping`。

# <a name="c"></a>[C#](#tab/c-sharp)

1. 创建一个新表，该表采用类似于 JSON 输入数据的架构。 我们将对下面的所有示例和引入命令使用此表。 

    ```csharp
    var table = "Events";
    var command =
        CslCommandGenerator.GenerateTableCreateCommand(
            table,
            new[]
            {
                Tuple.Create("Time", "System.DateTime"),
                Tuple.Create("Device", "System.String"),
                Tuple.Create("MessageId", "System.String"),
                Tuple.Create("Temperature", "System.Double"),
                Tuple.Create("Humidity", "System.Double"),
            });

    kustoClient.ExecuteControlCommand(command);
    ```

1. 创建 JSON 映射。

    ```csharp
    var tableMapping = "FlatEventMapping";
    var command =
         CslCommandGenerator.GenerateTableMappingCreateCommand(
            Data.Ingestion.IngestionMappingKind.Json,
            "",
            tableMapping,
            new[]
            {
               new ColumnMapping() {ColumnName = "Time", Properties = new Dictionary<string, string>() {{ MappingConsts.Path, "$.timestamp"} } },
               new ColumnMapping() {ColumnName = "Device", Properties = new Dictionary<string, string>() {{ MappingConsts.Path, "$.deviceId" } } },
               new ColumnMapping() {ColumnName = "MessageId", Properties = new Dictionary<string, string>() {{ MappingConsts.Path, "$.messageId" } } },
               new ColumnMapping() {ColumnName = "Temperature", Properties = new Dictionary<string, string>() {{ MappingConsts.Path, "$.temperature" } } },
               new ColumnMapping() { ColumnName= "Humidity", Properties = new Dictionary<string, string>() {{ MappingConsts.Path, "$.humidity" } } },
            });

    kustoClient.ExecuteControlCommand(command);
    ```

    在此映射中，根据表架构的定义，`timestamp` 条目将作为 `datetime` 数据类型引入到 `Time` 列。    

1. 将数据引入 `Events` 表中。

    ```csharp
    var blobPath = "https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.json,
            IngestionMapping = new IngestionMapping()
            {
                IngestionMappingReference = tableMapping
            }
        };

    ingestClient.IngestFromStorageAsync(blobPath, properties);
    ```

    文件“simple.json”包含几条行分隔的 JSON 记录。 格式为 `json`，在引入命令中使用的映射是创建的 `FlatEventMapping`。

# <a name="python"></a>[Python](#tab/python)

1. 创建一个新表，该表采用类似于 JSON 输入数据的架构。 我们将对下面的所有示例和引入命令使用此表。 

    ```python
    TABLE = "RawEvents"
    CREATE_TABLE_COMMAND = ".create table " + TABLE + " (Time: datetime, Device: string, MessageId: string, Temperature: double, Humidity: double)"
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_TABLE_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. 创建 JSON 映射。

    ```python
    MAPPING = "FlatEventMapping"
    CREATE_MAPPING_COMMAND = ".create table Events ingestion json mapping '" + MAPPING + """' '[{"column":"Time","Properties":{"path":"$.timestamp"}},{"column":"Device","Properties":{"path":"$.deviceId"}},{"column":"MessageId","Properties":{"path":"$.messageId"}},{"column":"Temperature","Properties":{"path":"$.temperature"}},{"column":"Humidity","Properties":{"path":"$.humidity"}}]'""" 
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_MAPPING_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. 将数据引入 `Events` 表中。

    ```python
    BLOB_PATH = 'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.json, mappingReference=MAPPING)
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

    文件“simple.json”包含几条行分隔的 JSON 记录。 格式为 `json`，在引入命令中使用的映射是创建的 `FlatEventMapping`。    
---

## <a name="ingest-multi-lined-json-records"></a>引入多行 JSON 记录

此示例引入多行 JSON 记录。 每个 JSON 属性映射到表中的单个列。 文件“multilined.json”包含几条缩进的 JSON 记录。 格式 `multijson` 告知引擎按 JSON 结构读取记录。

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

将数据引入 `Events` 表中。

```kusto
.ingest into table Events (h'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/multilined.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D') with '{"format":"multijson", "ingestionMappingReference":"FlatEventMapping"}'
```

# <a name="c"></a>[C#](#tab/c-sharp)

将数据引入 `Events` 表中。

```csharp
var tableMapping = "FlatEventMapping";
var blobPath = "https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/multilined.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
var properties =
    new KustoQueuedIngestionProperties(database, table)
    {
        Format = DataSourceFormat.multijson,
        IngestionMapping = new IngestionMapping()
        {
            IngestionMappingReference = tableMapping
        }
    };

ingestClient.IngestFromStorageAsync(blobPath, properties);
```

# <a name="python"></a>[Python](#tab/python)

将数据引入 `Events` 表中。

```python
MAPPING = "FlatEventMapping"
BLOB_PATH = 'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/multilined.JSON?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.multijson, mappingReference=MAPPING)
BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
INGESTION_CLIENT.ingest_from_blob(
    BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
```

---

## <a name="ingest-json-records-containing-arrays"></a>引入包含数组的 JSON 记录

数组数据类型是按顺序排列的值集合。 JSON 数组的引入由[更新策略](kusto/management/update-policy.md)来完成。 JSON 将按原样引入到中间表。 更新策略针对 `RawEvents` 表运行某个预定义的函数，并将结果重新引入到目标表。 我们将引入采用以下结构的数据：

```json
{
    "records": 
    [
        {
            "timestamp": "2019-05-02 15:23:50.0000000",
            "deviceId": "ddbc1bf5-096f-42c0-a771-bc3dca77ac71",
            "messageId": "7f316225-839a-4593-92b5-1812949279b3",
            "temperature": 31.0301639051317,
            "humidity": 62.0791099602725
        },
        {
            "timestamp": "2019-05-02 15:23:51.0000000",
            "deviceId": "ddbc1bf5-096f-42c0-a771-bc3dca77ac71",
            "messageId": "57de2821-7581-40e4-861e-ea3bde102364",
            "temperature": 33.7529423105311,
            "humidity": 75.4787976739364
        }
    ]
}
```

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

1. 使用 `mv-expand` 运算符创建一个 `update policy` 函数用于扩展 `records` 的集合，使集合中的每个值收到一个单独的行。 我们将使用表 `RawEvents` 作为源表，使用 `Events` 作为目标表。

    ```kusto
    .create function EventRecordsExpand() {
        RawEvents
        | mv-expand records = Event
        | project
            Time = todatetime(records["timestamp"]),
            Device = tostring(records["deviceId"]),
            MessageId = tostring(records["messageId"]),
            Temperature = todouble(records["temperature"]),
            Humidity = todouble(records["humidity"])
    }
    ```

1. 该函数收到的架构必须与目标表的架构相匹配。 使用 `getschema` 运算符检查架构。

    ```kusto
    EventRecordsExpand() | getschema
    ```

1. 将更新策略添加到目标表。 此策略将自动对 `RawEvents` 中间表中的任何新引入数据运行查询，并将结果引入到 `Events` 表中。 定义零保留期策略，以避免持久保存中间表。

    ```kusto
    .alter table Events policy update @'[{"Source": "RawEvents", "Query": "EventRecordsExpand()", "IsEnabled": "True"}]'
    ```

1. 将数据引入 `RawEvents` 表中。

    ```kusto
    .ingest into table RawEvents (h'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/array.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D') with '{"format":"multijson", "ingestionMappingReference":"RawEventMapping"}'
    ```

1. 检查 `Events` 表中的数据。

    ```kusto
    Events
    ```

# <a name="c"></a>[C#](#tab/c-sharp)

1. 使用 `mv-expand` 运算符创建一个 update 函数用于扩展 `records` 的集合，使集合中的每个值收到一个单独的行。 我们将使用表 `RawEvents` 作为源表，使用 `Events` 作为目标表。   

    ```csharp
    var command =
        CslCommandGenerator.GenerateCreateFunctionCommand(
            "EventRecordsExpand",
            "UpdateFunctions",
            string.Empty,
            null,
            @"RawEvents
                | mv-expand records = Event
                | project
                    Time = todatetime(records['timestamp']),
                    Device = tostring(records['deviceId']),
                    MessageId = tostring(records['messageId']),
                    Temperature = todouble(records['temperature']),
                    Humidity = todouble(records['humidity'])",
            ifNotExists: false);

    kustoClient.ExecuteControlCommand(command);
    ```

    > [!NOTE]
    > 该函数收到的架构必须与目标表的架构相匹配。

1. 将更新策略添加到目标表。 此策略将针对 `RawEvents` 中间表中任何新引入数据自动运行查询，并将查询结果引入到 `Events` 表中。 定义零保留期策略，以避免持久保存中间表。

    ```csharp
    var command =
        ".alter table Events policy update @'[{'Source': 'RawEvents', 'Query': 'EventRecordsExpand()', 'IsEnabled': 'True'}]";

    kustoClient.ExecuteControlCommand(command);
    ```

1. 将数据引入 `RawEvents` 表中。

    ```csharp
    var table = "RawEvents";
    var tableMapping = "RawEventMapping";
    var blobPath = "https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/array.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.multijson,
            IngestionMapping = new IngestionMapping()
            {
                IngestionMappingReference = tableMapping
            }
        };

    ingestClient.IngestFromStorageAsync(blobPath, properties);
    ```
    
1. 检查 `Events` 表中的数据。

# <a name="python"></a>[Python](#tab/python)

1. 使用 `mv-expand` 运算符创建一个 update 函数用于扩展 `records` 的集合，使集合中的每个值收到一个单独的行。 我们将使用表 `RawEvents` 作为源表，使用 `Events` 作为目标表。   

    ```python
    CREATE_FUNCTION_COMMAND = 
        '''.create function EventRecordsExpand() { 
            RawEvents
            | mv-expand records = Event
            | project
                Time = todatetime(records["timestamp"]),
                Device = tostring(records["deviceId"]),
                MessageId = tostring(records["messageId"]),
                Temperature = todouble(records["temperature"]),
                Humidity = todouble(records["humidity"])
            }'''
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_FUNCTION_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

    > [!NOTE]
    > 该函数收到的架构必须与目标表的架构相匹配。

1. 将更新策略添加到目标表。 此策略将针对 `RawEvents` 中间表中任何新引入数据自动运行查询，并将查询结果引入到 `Events` 表中。 定义零保留期策略，以避免持久保存中间表。

    ```python
    CREATE_UPDATE_POLICY_COMMAND = 
        """.alter table Events policy update @'[{'Source': 'RawEvents', 'Query': 'EventRecordsExpand()', 'IsEnabled': 'True'}]"""
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_UPDATE_POLICY_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. 将数据引入 `RawEvents` 表中。

    ```python
    TABLE = "RawEvents"
    MAPPING = "RawEventMapping"
    BLOB_PATH = 'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/array.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.multijson, mappingReference=MAPPING)
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

1. 检查 `Events` 表中的数据。

---    

## <a name="ingest-json-records-containing-dictionaries"></a>引入包含字典的 JSON 记录

字典结构化 JSON 包含键值对。 JSON 记录使用 `JsonPath` 中的逻辑表达式经历引入映射过程。 可以引入采用以下结构的数据：

```json
{
    "event": 
    [
        {
            "Key": "timestamp",
            "Value": "2019-05-02 15:23:50.0000000"
        },
        {
            "Key": "deviceId",
            "Value": "ddbc1bf5-096f-42c0-a771-bc3dca77ac71"
        },
        {
            "Key": "messageId",
            "Value": "7f316225-839a-4593-92b5-1812949279b3"
        },
        {
            "Key": "temperature",
            "Value": 31.0301639051317
        },
        {
            "Key": "humidity",
            "Value": 62.0791099602725
        }
    ]
}
```

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

1. 创建 JSON 映射。

    ```kusto
    .create table Events ingestion json mapping 'KeyValueEventMapping' '[{"column":"a","Properties":{"path":"$.event[?(@.Key == \'timestamp\')]"}},{"column":"b","Properties":{"path":"$.event[?(@.Key == \'deviceId\')]"}},{"column":"c","Properties":{"path":"$.event[?(@.Key == \'messageId\')]"}},{"column":"d","Properties":{"path":"$.event[?(@.Key == \'temperature\')]"}},{"column":"Humidity","datatype":"string","Properties":{"path":"$.event[?(@.Key == \'humidity\')]"}}]'
    ```

1. 将数据引入 `Events` 表中。

    ```kusto
    .ingest into table Events h'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/dictionary.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=multijson, jsonMappingReference=KeyValueEventMapping)
    ```

# <a name="c"></a>[C#](#tab/c-sharp)

1. 创建 JSON 映射。

    ```csharp
    var tableName = "Events";
    var tableMapping = "KeyValueEventMapping";
    var command =
         CslCommandGenerator.GenerateTableMappingCreateCommand(
            Data.Ingestion.IngestionMappingKind.Json,
            "",
            tableMapping,
            new[]
            {
                new ColumnMapping() { ColumnName = "Time", Properties = new Dictionary<string, string>() { {
                    MappingConsts.Path,
                    "$.event[?(@.Key == 'timestamp')]"
                } } },
                    new ColumnMapping() { ColumnName = "Device", Properties = new Dictionary<string, string>() { {
                    MappingConsts.Path,
                    "$.event[?(@.Key == 'deviceId')]"
                } } }, new ColumnMapping() { ColumnName = "MessageId", Properties = new Dictionary<string, string>() { {
                    MappingConsts.Path,
                    "$.event[?(@.Key == 'messageId')]"
                } } }, new ColumnMapping() { ColumnName = "Temperature", Properties = new Dictionary<string, string>() { {
                    MappingConsts.Path,
                    "$.event[?(@.Key == 'temperature')]"
                } } }, new ColumnMapping() { ColumnName = "Humidity", Properties = new Dictionary<string, string>() { {
                    MappingConsts.Path,
                    "$.event[?(@.Key == 'humidity')]"
                } } },
            });

    kustoClient.ExecuteControlCommand(command);
    ```

1. 将数据引入 `Events` 表中。

    ```csharp
    var blobPath = "https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/dictionary.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.multijson,
            IngestionMapping = new IngestionMapping()
            {
                IngestionMappingReference = tableMapping
            }
        };
    ingestClient.IngestFromStorageAsync(blobPath, properties);
    ```

# <a name="python"></a>[Python](#tab/python)

1. 创建 JSON 映射。

    ```python
    MAPPING = "KeyValueEventMapping"
    CREATE_MAPPING_COMMAND = ".create table Events ingestion json mapping '" + MAPPING + """' '[{"column":"Time","path":"$.event[?(@.Key == 'timestamp')]"},{"column":"Device","path":"$.event[?(@.Key == 'deviceId')]"},{"column":"MessageId","path":"$.event[?(@.Key == 'messageId')]"},{"column":"Temperature","path":"$.event[?(@.Key == 'temperature')]"},{"column":"Humidity","path":"$.event[?(@.Key == 'humidity')]"}]'""" 
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_MAPPING_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. 将数据引入 `Events` 表中。

     ```python
    MAPPING = "KeyValueEventMapping"
    BLOB_PATH = 'https://kustosamplefiles.blob.core.chinaclouapi.cn/jsonsamplefiles/dictionary.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.multijson, mappingReference=MAPPING)u
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

---    

## <a name="next-steps"></a>后续步骤

* [数据引入概述](ingest-data-overview.md)
* [编写查询](write-queries.md)