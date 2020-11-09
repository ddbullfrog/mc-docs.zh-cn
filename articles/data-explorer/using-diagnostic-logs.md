---
title: 使用诊断日志监视 Azure 数据资源管理器的引入、命令和查询
description: 了解如何设置诊断日志，使 Azure 数据资源管理器能够监视引入、命令和查询操作。
author: orspod
ms.author: v-tawe
ms.reviewer: guregini
ms.service: data-explorer
ms.topic: how-to
origin.date: 09/18/2019
ms.date: 09/30/2020
ms.openlocfilehash: 42ceed6ecee864d9f71f02df0cb43c3e92df16a5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105691"
---
# <a name="monitor-azure-data-explorer-ingestion-commands-and-queries-using-diagnostic-logs"></a>使用诊断日志监视 Azure 数据资源管理器的引入、命令和查询

Azure 数据资源管理器是一项快速、完全托管的数据分析服务，用于实时分析从应用程序、网站和 IoT 设备等资源流式传输的海量数据。 [Azure Monitor 诊断日志](/azure/azure-monitor/platform/diagnostic-logs-overview)提供有关 Azure 资源操作的数据。 Azure 数据资源管理器使用诊断日志获取有关引入成功、引入失败、命令和查询操作的见解。 可将操作日志导出到 Azure 存储、事件中心或 Log Analytics 以监视引入、命令和查询状态。 可将 Azure 存储和 Azure 事件中心的日志路由到 Azure 数据资源管理器群集中的某个表，以进一步分析。

> [!IMPORTANT] 
> 诊断日志数据可能包含敏感数据。 请根据监视需求限制日志目标的权限。 

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请创建一个[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。
* 登录到 [Azure 门户](https://portal.azure.cn/)。
* 创建[群集和数据库](create-cluster-database-portal.md)。

## <a name="set-up-diagnostic-logs-for-an-azure-data-explorer-cluster"></a>设置 Azure 数据资源管理器群集的诊断日志

诊断日志可用于配置以下日志数据的收集：

# <a name="ingestion"></a>[引流](#tab/ingestion)

* **成功的引入操作** ：这些日志包含有关已成功完成的引入操作的信息。
* **失败的引入操作** ：这些日志包含有关失败的引入操作的详细信息，包括错误详细信息。 

# <a name="commands-and-queries"></a>[命令和查询](#tab/commands-and-queries)

* **命令** ：这些日志包含已达到最终状态的管理命令的相关信息。
* **查询** ：这些日志包含已达到最终状态的查询的相关详细信息。 

    > [!NOTE]
    > 查询日志数据不包含查询文本。

---

然后可根据规范将数据存档到存储帐户、流式传输到事件中心，或发送到 Log Analytics。

### <a name="enable-diagnostic-logs"></a>启用诊断日志

诊断日志默认已禁用。 若要启用诊断日志，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.cn)中，选择要监视的 Azure 数据资源管理器群集资源。
1. 在“监视”下，选择“诊断设置”   。
  
    ![添加诊断日志](media/using-diagnostic-logs/add-diagnostic-logs.png)

1. 选择“添加诊断设置”。 
1. 在“诊断设置”窗口中： 

    :::image type="content" source="media/using-diagnostic-logs/configure-diagnostics-settings.png" alt-text="配置诊断设置":::

    1. 选择诊断设置的 **名称** 。
    1. 选择一个或多个目标：存储帐户、事件中心或 Log Analytics。
    1. 选择要收集的日志：`SucceededIngestion`、`FailedIngestion`、`Command` 或 `Query`。
    1. 选择要收集的[指标](using-metrics.md#supported-azure-data-explorer-metrics)（可选）。  
    1. 选择“保存”以保存新的诊断日志设置和指标。 

在几分钟内即会完成新的设置。 日志随后会显示在配置的存档目标（存储帐户、事件中心或 Log Analytics）中。 

> [!NOTE]
> 如果将日志发送到 Log Analytics，则 `SucceededIngestion`、`FailedIngestion`、`Command` 和 `Query` 日志将分别存储在名为 `SucceededIngestion`、`FailedIngestion`、`ADXCommand` 和 `ADXQuery` 的 Log Analytics 表中。

## <a name="diagnostic-logs-schema"></a>诊断日志架构

所有 [Azure Monitor 诊断日志共享一个通用的顶级架构](/azure-monitor/platform/diagnostic-logs-schema)。 Azure 数据资源管理器对其自身的事件使用唯一属性。 所有日志均以 JSON 格式存储。

# <a name="ingestion"></a>[引流](#tab/ingestion)

### <a name="ingestion-logs-schema"></a>引入日志架构

日志 JSON 字符串包含下表中列出的元素：

|名称               |说明
|---                |---
|time               |报告时间
|ResourceId         |Azure Resource Manager 资源 ID
|operationName      |操作名称：'MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION'
|operationVersion   |架构版本：'1.0' 
|category           |操作类别。 `SucceededIngestion` 或 `FailedIngestion`。 [成功的操作](#successful-ingestion-operation-log)或[失败的操作](#failed-ingestion-operation-log)的属性不同。
|properties         |操作的详细信息。

#### <a name="successful-ingestion-operation-log"></a>成功引入操作日志

**示例：**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "SucceededIngestion",
    "properties":
    {
        "succeededOn": "2019-05-27 07:55:05.3693628",
        "operationId": "b446c48f-6e2f-4884-b723-92eb6dc99cc9",
        "database": "Samples",
        "table": "StormEvents",
        "ingestionSourceId": "66a2959e-80de-4952-975d-b65072fc571d",
        "ingestionSourcePath": "https://kustoingestionlogs.blob.core.chinacloudapi.cn/sampledata/events8347293.json",
        "rootActivityId": "d0bd5dd3-c564-4647-953e-05670e22a81d"
    }
}
```
**成功操作诊断日志的属性**

|名称               |说明
|---                |---
|succeededOn        |引入完成时间
|operationId        |Azure 数据资源管理器引入操作 ID
|database           |目标数据库的名称
|表              |目标表的名称
|ingestionSourceId  |引入数据源的 ID
|ingestionSourcePath|引入数据源或 Blob URI 的路径
|rootActivityId     |活动 ID

#### <a name="failed-ingestion-operation-log"></a>失败引入操作日志

**示例：**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "FailedIngestion",
    "properties":
    {
        "failedOn": "2019-05-27 08:57:05.4273524",
        "operationId": "5956515d-9a48-4544-a514-cf4656fe7f95",
        "database": "Samples",
        "table": "StormEvents",
        "ingestionSourceId": "eee56f8c-2211-4ea4-93a6-be556e853e5f",
        "ingestionSourcePath": "https://kustoingestionlogs.blob.core.chinacloudapi.cn/sampledata/events5725592.json",
        "rootActivityId": "52134905-947a-4231-afaf-13d9b7b184d5",
        "details": "Permanent failure downloading blob. URI: ..., permanentReason: Download_SourceNotFound, DownloadFailedException: 'Could not find file ...'",
        "errorCode": "Download_SourceNotFound",
        "failureStatus": "Permanent",
        "originatesFromUpdatePolicy": false,
        "shouldRetry": false
    }
}
```

**失败操作诊断日志的属性**

|名称               |说明
|---                |---
|failedOn           |引入完成时间
|operationId        |Azure 数据资源管理器引入操作 ID
|database           |目标数据库的名称
|表              |目标表的名称
|ingestionSourceId  |引入数据源的 ID
|ingestionSourcePath|引入数据源或 Blob URI 的路径
|rootActivityId     |活动 ID
|详细信息            |失败和错误消息的详细说明
|errorCode          |错误代码 
|failureStatus      |`Permanent` 或 `Transient`。 重试暂时性故障可能会成功。
|originatesFromUpdatePolicy|如果故障源自更新策略，则为 True
|shouldRetry        |如果重试可以成功，则为 True

# <a name="commands-and-queries"></a>[命令和查询](#tab/commands-and-queries)

### <a name="commands-and-queries-logs-schema"></a>命令和查询日志架构

日志 JSON 字符串包含下表中列出的元素：

|名称               |说明
|---                |---
|time               |报告时间
|ResourceId         |Azure Resource Manager 资源 ID
|operationName      |操作名称：“MICROSOFT.KUSTO/CLUSTERS/COMMAND/ACTION”或“MICROSOFT.KUSTO/CLUSTERS/QUERY/ACTION”。 命令和查询日志的属性不同。
|operationVersion   |架构版本：'1.0' 
|category           |操作类别：`Command` 或 `Query`。 命令和查询日志的属性不同。
|properties         |操作的详细信息。

#### <a name="command-log"></a>命令日志

**示例：**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/COMMAND/ACTION",
    "operationVersion": "1.0",
    "category": "Command",
    "properties":
    {
        "RootActivityId": "d0bd5dd3-c564-4647-953e-05670e22a81d",
        "StartedOn": "2020-09-12T18:06:04.6898353Z",
        "LastUpdatedOn": "2020-09-12T18:06:04.6898353Z",
        "Database": "DatabaseSample",
        "State": "Completed",
        "FailureReason": "XXX",
        "TotalCpu": "00:01:30.1562500",
        "CommandType": "ExtentsDrop",
        "Application": "Kusto.WinSvc.DM.Svc",
        "ResourceUtilization": "{\"CacheStatistics\":{\"Memory\":{\"Hits\":0,\"Misses\":0},\"Disk\":{\"Hits\":0,\"Misses\":0},\"Shards\":{\"Hot\":{\"HitBytes\":0,\"MissBytes\":0,\"RetrieveBytes\":0},\"Cold\":{\"HitBytes\":0,\"MissBytes\":0,\"RetrieveBytes\":0},\"BypassBytes\":0}},\"TotalCpu\":\"00:00:00\",\"MemoryPeak\":0,\"ScannedExtentsStatistics\":{\"MinDataScannedTime\":null,\"MaxDataScannedTime\":null,\"TotalExtentsCount\":0,\"ScannedExtentsCount\":0,\"TotalRowsCount\":0,\"ScannedRowsCount\":0}}",
        "Duration": "00:03:30.1562500",
        "User": "AAD app id=0571b364-eeeb-4f28-ba74-90a8b4132b53",
        "Principal": "aadapp=0571b364-eeeb-4f28-ba74-90a3b4136b53;5c443533-c927-4410-a5d6-4d6a5443b64f"
    }
}
```
**命令诊断日志的属性**

|名称               |说明
|---                |---
|RootActivityId |根活动 ID
|StartedOn        |该命令开始的时间 (UTC)
|LastUpdatedOn        |该命令结束的时间 (UTC)
|数据库          |运行命令的数据库的名称
|状态              |命令结束的状态
|FailureReason  |失败原因
|TotalCpu |CPU 总持续时间
|CommandType     |命令类型
|应用程序     |调用了命令的应用程序的名称
|ResourceUtilization     |命令资源利用率
|Duration     |命令持续时间
|用户     |调用了查询的用户
|主体     |调用了查询的主体

#### <a name="query-log"></a>查询日志

**示例：**

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/QUERY/ACTION",
    "operationVersion": "1.0",
    "category": "Query",
    "properties": {
        "RootActivityId": "3e6e8814-e64f-455a-926d-bf16229f6d2d",
        "StartedOn": "2020-09-04T13:45:45.331925Z",
        "LastUpdatedOn": "2020-09-04T13:45:45.3484372Z",
        "Database": "DatabaseSample",
        "State": "Completed",
        "FailureReason": "[none]",
        "TotalCpu": "00:00:00",
        "ApplicationName": "MyApp",
        "MemoryPeak": 0,
        "Duration": "00:00:00.0165122",
        "User": "AAD app id=0571b364-eeeb-4f28-ba74-90a8b4132b53",
        "Principal": "aadapp=0571b364-eeeb-4f28-ba74-90a8b4132b53;5c823e4d-c927-4010-a2d8-6dda2449b6cf",
        "ScannedExtentsStatistics": {
            "MinDataScannedTime": "2020-07-27T08:34:35.3299941",
            "MaxDataScannedTime": "2020-07-27T08:34:41.991661",
            "TotalExtentsCount": 64,
            "ScannedExtentsCount": 64,
            "TotalRowsCount": 320,
            "ScannedRowsCount": 320
        },
        "CacheStatistics": {
            "Memory": {
                "Hits": 192,
                "Misses": 0
          },
            "Disk": {
                "Hits": 0,
                "Misses": 0
      },
            "Shards": {
                "Hot": {
                    "HitBytes": 0,
                    "MissBytes": 0,
                    "RetrieveBytes": 0
        },
                "Cold": {
                    "HitBytes": 0,
                    "MissBytes": 0,
                    "RetrieveBytes": 0
        },
            "BypassBytes": 0
      }
    },
    "ResultSetStatistics": {
        "TableCount": 1,
        "TablesStatistics": [
        {
          "RowCount": 1,
          "TableSize": 9
        }
      ]
    }
  }
}
```

**查询诊断日志的属性**

|名称               |说明
|---                |---
|RootActivityId |根活动 ID
|StartedOn        |该命令开始的时间 (UTC)
|LastUpdatedOn           |该命令结束的时间 (UTC)
|数据库              |运行命令的数据库的名称
|状态  |命令结束的状态
|FailureReason|失败原因
|TotalCpu     |CPU 总持续时间
|ApplicationName            |调用了查询的应用程序的名称
|MemoryPeak          |内存峰值
|Duration      |命令持续时间
|用户|调用了查询的用户
|主体        |调用了查询的主体
|ScannedExtentsStatistics        | 包含扫描的盘区统计信息
|MinDataScannedTime        |最短数据扫描时间
|MaxDataScannedTime        |最长数据扫描时间
|TotalExtentsCount        |盘区总数
|ScannedExtentsCount        |扫描的盘区计数
|TotalRowsCount        |总行计数
|ScannedRowsCount        |扫描的行计数
|CacheStatistics        |包含缓存统计信息
|内存        |包含缓存内存统计信息
|命中数        |内存缓存命中数
|未命中数        |内存缓存未命中数
|磁盘        |包含缓存磁盘统计信息
|命中数        |磁盘缓存命中数
|未命中数        |磁盘缓存未命中数
|Shards        |包含热和冷分片缓存统计信息
|热        |包含热分片缓存统计信息
|HitBytes        |分片热缓存命中数
|MissBytes        |分片热缓存未命中数
|RetrieveBytes        | 分片热缓存检索的字节数
|冷        |包含冷分片缓存统计信息
|HitBytes        |分片冷缓存命中数
|MissBytes        |分片冷缓存未命中数
|RetrieveBytes        |分片冷缓存检索的字节数
|BypassBytes        |分片缓存绕过字节数
|ResultSetStatistics        |包含结果集统计信息
|TableCount        |结果集表计数
|TablesStatistics        |包含结果集表统计信息
|RowCount        | 结果集表行计数
|TableSize        |结果集表行计数

---

## <a name="next-steps"></a>后续步骤

* [使用指标来监视群集运行状况](using-metrics.md)
* [教程：在 Azure 数据资源管理器中引入和查询监视数据](ingest-data-no-code.md)，可帮助获取引入诊断日志
