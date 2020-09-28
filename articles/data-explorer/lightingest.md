---
title: 使用 LightIngest 将数据引入 Azure 数据资源管理器。
description: 了解 LightIngest - 用于将即席数据引入 Azure 数据资源管理器的命令行实用工具。
author: orspod
ms.author: v-tawe
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: how-to
origin.date: 06/28/2020
ms.date: 09/24/2020
ms.openlocfilehash: a6f0a66e758a2b3781b4c182e85bfe339098152a
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146359"
---
# <a name="use-lightingest-to-ingest-data-to-azure-data-explorer"></a>使用 LightIngest 将数据引入 Azure 数据资源管理器
 
LightIngest 是用于将即席数据引入 Azure 数据资源管理器的命令行实用工具。 该实用工具可以从本地文件夹或 Azure Blob 存储容器提取源数据。
当你想要引入大量数据时，LightIngest 最有用，因为引入持续时间没有时间限制。 当你想要以后根据记录的创建时间而不是引入时间来查询记录时，它也很有用。

## <a name="prerequisites"></a>先决条件

* LightIngest - 将其作为 [Microsoft.Azure.Kusto.Tools NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/)的一部分下载

    :::image type="content" source="media/lightingest/lightingest-download-area.png" alt-text="Lightingest 下载":::

* WinRAR - 从 [www.win-rar.com/download.html](http://www.win-rar.com/download.html) 下载

## <a name="install-lightingest"></a>安装 LightIngest

1. 在计算机上导航到 LightIngest 所下载到的位置。
1. 使用 WinRAR 将 tools 目录解压缩到计算机上。**

## <a name="run-lightingest"></a>运行 LightIngest

1. 在计算机上导航到解压缩的 tools 目录。**
1. 从位置栏中删除现有的位置信息。
    
    :::image type="content" source="kusto/tools/images/KustoTools-Lightingest/lightingest-locationbar.png" alt-text="删除 LightIngest 的现有位置信息":::

1. 输入 `cmd`，然后按 Enter。****
1. 在命令提示符下，输入 `LightIngest.exe`，后接相关的命令行参数。

    > [!Tip]
    > 若要查看支持的命令行参数列表，请输入 `LightIngest.exe /help`。
    >
    > :::image type="content" source="media/lightingest/lightingest-cmd-line-help.png" alt-text="LightIngest 的命令行帮助":::

1. 输入 `ingest-`，后接用于管理引入操作的 Azure 数据资源管理器群集的连接字符串。
    将连接字符串括在双引号中，并遵循 [Kusto 连接字符串规范](kusto/api/connection-strings/kusto.md)。

    例如：

    ```
    ingest-{Cluster name and region}.kusto.chinacloudapi.cn;AAD Federated Security=True"  -db:{Database} -table:Trips -source:"https://{Account}.blob.core.chinacloudapi.cn/{ROOT_CONTAINER};{StorageAccountKey}" -pattern:"*.csv.gz" -format:csv -limit:2 -ignoreFirst:true -cr:10.0 -dontWait:true
    ```

### <a name="recommendations"></a>建议

* 建议的方法是让 LightIngest 使用 `https://ingest-{yourClusterNameAndRegion}.kusto.chinacloudapi.cn` 上的引入终结点。 这样，Azure 数据资源管理器服务就可以管理引入负载，而你可以在发生暂时性错误时轻松恢复。 不过，也可以将 LightIngest 配置为直接用于引擎终结点 (`https://{yourClusterNameAndRegion}.kusto.chinacloudapi.cn`)。

   > [!NOTE]
   > 如果直接与引擎终结点一起引入，则不需要包括 `ingest-`。 但是，不会有 DM 功能来保护引擎和提高引入成功率。

* 若要提供最佳引入性能，需要原始数据大小，以便 LightIngest 可以估计本地文件的未压缩大小。 但如果没有事先下载已压缩的 Blob，LightIngest 可能无法正确估算这些 Blob 的原始大小。 因此，在引入压缩的 Blob 时，请将 Blob 元数据的 `rawSizeBytes` 属性设置为以字节为单位的非压缩数据大小。

## <a name="command-line-arguments"></a>命令行参数

|参数名称            |类型     |说明       |必需/可选
|------------------------------|--------|----------|-----------------------------|
|                               |string   |[Azure 数据资源管理器连接字符串](kusto/api/connection-strings/kusto.md)，指定用于处理引入操作的 Kusto 终结点。 应括在双引号中 | 必需
|-database、-db          |string  |目标 Azure 数据资源管理器数据库名称 | 可选  |
|-table                  |string  |目标 Azure 数据资源管理器表名称 | 必需 |
|-sourcePath、-source      |string  |源文件的路径，或 Blob 容器的根 URI。 如果数据位于 Blob 中，则必须包含存储帐户密钥或 SAS。 建议括在双引号中 |必需 |
|-prefix                  |string  |当要引入的源数据驻留在 Blob 存储中时，此 URL 前缀（不包括容器名称）将由所有 Blob 共享。 <br>例如，如果数据位于 `MyContainer/Dir1/Dir2` 中，则前缀应是 `Dir1/Dir2`。 建议括在双引号中 | 可选  |
|-pattern        |string  |提取源文件/Blob 时所遵循的模式。 支持通配符。 例如，`"*.csv"`。 建议括在双引号中 | 可选  |
|-zipPattern     |string  |选择要引入 ZIP 存档中的哪些文件时要使用的正则表达式。<br>存档中的所有其他文件会被忽略。 例如，`"*.csv"`。 建议括在双引号中 | 可选  |
|-format、-f           |string  | 源数据格式。 必须是[支持的格式](ingestion-supported-formats.md)之一 | 可选  |
|-ingestionMappingPath、-mappingPath |string  |用于引入列映射的本地文件路径。 对于 Json 和 Avro 格式是必需的。 请参阅[数据映射](kusto/management/mappings.md) | 可选  |
|-ingestionMappingRef、-mappingRef  |string  |之前在表上创建的引入列映射的名称。 对于 Json 和 Avro 格式是必需的。 请参阅[数据映射](kusto/management/mappings.md) | 可选  |
|-creationTimePattern      |string  |如果设置此参数，它将用于从文件或 Blob 路径中提取 CreationTime 属性。 请参阅[如何使用 `CreationTime` 引入数据](#how-to-ingest-data-using-creationtime) |可选  |
|-ignoreFirstRow、-ignoreFirst |bool    |如果设置此参数，将忽略每个文件/Blob 的第一条记录（例如，如果源数据包含标头） | 可选  |
|-tag            |string   |要与引入的数据相关联的[标记](kusto/management/extents-overview.md#extent-tagging)。 允许多个标记 | 可选  |
|-dontWait           |bool     |如果设置为“true”，则不等待引入完成。 引入大量文件/Blob 时非常有用 |可选  |
|-compression、-cr          |Double |压缩比提示。 引入压缩的文件/Blob 来帮助 Azure 数据资源管理器评估原始数据大小时，此参数非常有用。 计算方式为原始大小除以压缩大小 |可选  |
|-limit、-l           |integer   |如果设置此参数，则会将引入限制为前 N 个文件 |可选  |
|-listOnly、-list        |bool    |如果设置此参数，仅显示已选择引入的项| 可选  |
|-ingestTimeout   |integer  |完成所有引入操作的超时（分钟）。 默认为 `60`| 可选  |
|-forceSync        |bool  |如果设置此参数，将强制同步引入。 默认为 `false` |可选  |
|-dataBatchSize        |integer  |设置每个引入操作的总大小限制（MB，非压缩） |可选  |
|-filesInBatch            |integer |设置每个引入操作的文件/Blob 计数限制 |可选  |
|-devTracing、-trace       |string    |如果设置此参数，诊断日志将写入本地目录（默认为当前目录中的 `RollingLogs`，可以通过设置开关值修改此位置） | 可选  |

## <a name="azure-blob-specific-capabilities"></a>特定于 Azure Blob 的功能

与 Azure Blob 配合使用时，LightIngest 将使用特定的 Blob 元数据属性来增强引入过程。

|元数据属性                            | 使用情况                                                                           |
|---------------------------------------------|---------------------------------------------------------------------------------|
|`rawSizeBytes`, `kustoUncompressedSizeBytes` | 如果设置此参数，它将解释为非压缩数据大小                       |
|`kustoCreationTime`, `kustoCreationTimeUtc`  | 解释为 UTC 时间戳。 如果设置此参数，它将用于重写 Kusto 中的创建时间。 在回填方案中非常有用 |

## <a name="usage-examples"></a>用法示例

<!-- Waiting for Tzvia or Vladik to rewrite the instructions for this example before publishing it

### Ingesting a specific number of blobs in JSON format

* Ingest two blobs under a specified storage account {Account}, in `JSON` format matching the pattern `.json`
* Destination is the database {Database}, the table `SampleData`
* Indicate that your data is compressed with the approximate ratio of 10.0
* LightIngest won't wait for the ingestion to be completed

To use the LightIngest command below:
1. Create a table command and enter the table name into the LightIngest command, replacing `SampleData`.
1. Create a mapping command and enter the IngestionMappingRef command, replacing `SampleData_mapping`.
1. Copy your cluster name and enter it into the LightIngest command, replacing `{ClusterandRegion}`.
1. Enter the database name into the LightIngest command, replacing `{Database name}`.
1. Replace `{Account}` with your account name and replace `{ROOT_CONTAINER}?{SAS token}` with the appropriate information.

    ```
    LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.chinacloudapi.cn;Fed=True"  
        -db:{Database name} 
        -table:SampleData 
        -source:"https://{Account}.blob.core.chinacloudapi.cn/{ROOT_CONTAINER}?{SAS token}" 
        -IngestionMappingRef:SampleData_mapping 
        -pattern:"*.json" 
        -format:JSON 
        -limit:2 
        -cr:10.0 
        -dontWait:true
    ```
     
1. In Azure Data Explorer, open query count.

    ![Ingestion result in Azure Data Explorer](media/lightingest/lightingest-show-failure-count.png)
-->

### <a name="how-to-ingest-data-using-creationtime"></a>如何使用 CreationTime 引入数据

将历史数据从现有系统加载到 Azure 数据资源管理器时，所有记录接收相同的引入日期。 若要启用按创建时间而不是引入时间对数据进行分区，可以使用 `-creationTimePattern` 参数。 `-creationTimePattern` 参数从文件或 Blob 路径中提取 `CreationTime` 属性。 此模式不需要反映整个项路径，只需反映包含要使用的时间戳的部分。

参数值必须包括：
* 紧邻在时间戳格式前面的常量文本，用单引号括起来（前缀）
* 时间戳格式，采用标准的 [.NET 日期时间表示法](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)
* 紧跟在时间戳之后的常量文本（后缀）。

**示例** 

* 包含如下日期时间的 Blob 名称：`historicalvalues19840101.parquet`（时间戳中年份以四位数表示，月份以两位数表示，日期以两位数表示） 
    
    `-creationTimePattern` 参数的值是文件名的一部分：“'historicalvalues'yyyyMMdd'.parquet'”

    ```kusto
    ingest-{Cluster name and region}.kusto.chinacloudapi.cn;AAD Federated Security=True -db:{Database} -table:Trips -source:"https://{Account}.blob.core.chinacloudapi.cn/{ROOT_CONTAINER};{StorageAccountKey}" -creationTimePattern:"'historicalvalues'yyyyMMdd'.parquet'"
     -pattern:"*.csv.gz" -format:csv -limit:2 -ignoreFirst:true -cr:10.0 -dontWait:true
    ```

* 一个引用分层文件夹结构的 Blob URI（如 `https://storageaccount/container/folder/2002/12/01/blobname.extension`） 

    `-creationTimePattern` 参数的值为文件夹结构的一部分：“'folder/'yyyy/MM/dd'/blob'”

   ```kusto
    ingest-{Cluster name and region}.kusto.chinacloudapi.cn;AAD Federated Security=True -db:{Database} -table:Trips -source:"https://{Account}.blob.core.chinacloudapi.cn/{ROOT_CONTAINER};{StorageAccountKey}" -creationTimePattern:"'folder/'yyyy/MM/dd'/blob'"
     -pattern:"*.csv.gz" -format:csv -limit:2 -ignoreFirst:true -cr:10.0 -dontWait:true
    ```

### <a name="ingesting-blobs-using-a-storage-account-key-or-a-sas-token"></a>使用存储帐户密钥或 SAS 令牌引入 Blob

* 引入指定存储帐户 `ACCOUNT` 下文件夹 `DIR` 中容器 `CONT` 下的与模式 `*.csv.gz` 匹配的 10 个 Blob
* 目标是数据库 `DB` 表 `TABLE`，引入映射 `MAPPING` 已在目标中预先创建
* 工具将等到引入操作完成
* 请注意，有不同的选项用于指定目标数据库和存储帐户密钥与SAS 令牌

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.chinacloudapi.cn;Fed=True"
  -database:DB
  -table:TABLE
  -source:"https://ACCOUNT.blob.core.chinacloudapi.cn/{ROOT_CONTAINER};{StorageAccountKey}"
  -prefix:"DIR"
  -pattern:*.csv.gz
  -format:csv
  -mappingRef:MAPPING
  -limit:10

LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.chinacloudapi.cn;Fed=True;Initial Catalog=DB"
  -table:TABLE
  -source:"https://ACCOUNT.blob.core.chinacloudapi.cn/{ROOT_CONTAINER}?{SAS token}"
  -prefix:"DIR"
  -pattern:*.csv.gz
  -format:csv
  -mappingRef:MAPPING
  -limit:10
```

### <a name="ingesting-all-blobs-in-a-container-not-including-header-rows"></a>引入容器中的所有 Blob，不包括标头行

* 引入指定存储帐户 `ACCOUNT` 下文件夹 `DIR1/DIR2` 中容器 `CONT` 下的与模式 `*.csv.gz` 匹配的所有 Blob
* 目标是数据库 `DB` 表 `TABLE`，引入映射 `MAPPING` 已在目标中预先创建
* 源 Blob 包含标头行，因此工具被指示删除每个 Blob 的第一条记录
* 工具将发布要引入的数据，且不会等待引入操作完成

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.chinacloudapi.cn;Fed=True"
  -database:DB
  -table:TABLE
  -source:"https://ACCOUNT.blob.core.chinacloudapi.cn/{ROOT_CONTAINER}?{SAS token}"
  -prefix:"DIR1/DIR2"
  -pattern:*.csv.gz
  -format:csv
  -mappingRef:MAPPING
  -ignoreFirstRow:true
```

### <a name="ingesting-all-json-files-from-a-path"></a>从一个路径引入所有 JSON 文件

* 引入路径 `PATH` 下的与模式 `*.json` 匹配的所有文件
* 目标为数据库 `DB` 表 `TABLE`，引入映射已在本地文件 `MAPPING_FILE_PATH` 中定义
* 工具将发布要引入的数据，且不会等待引入操作完成

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.chinacloudapi.cn;Fed=True"
  -database:DB
  -table:TABLE
  -source:"PATH"
  -pattern:*.json
  -format:json
  -mappingPath:"MAPPING_FILE_PATH"
```

### <a name="ingesting-files-and-writing-diagnostic-trace-files"></a>引入文件并写入诊断跟踪文件

* 引入路径 `PATH` 下的与模式 `*.json` 匹配的所有文件
* 目标为数据库 `DB` 表 `TABLE`，引入映射已在本地文件 `MAPPING_FILE_PATH` 中定义
* 工具将发布要引入的数据，且不会等待引入操作完成
* 诊断跟踪文件将在本地写入到文件夹 `LOGS_PATH` 下

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.chinacloudapi.cn;Fed=True"
  -database:DB
  -table:TABLE
  -source:"PATH"
  -pattern:*.json
  -format:json
  -mappingPath:"MAPPING_FILE_PATH"
  -trace:"LOGS_PATH"
```
