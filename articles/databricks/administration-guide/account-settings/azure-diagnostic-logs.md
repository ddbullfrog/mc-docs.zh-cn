---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/04/2020
title: Azure Databricks 中的诊断日志记录 - Azure Databricks
description: 了解如何配置由 Azure Databricks 用户执行的活动的诊断日志记录，使你的企业可以监视详细的 Azure Databricks 使用模式。
ms.openlocfilehash: f2db8b8a8c1e87278168987258062eba8efe3cf9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106657"
---
# <a name="diagnostic-logging-in-azure-databricks"></a><a id="azure-diagnostic-logging"> </a><a id="diagnostic-logging-in-azure-databricks"> </a>Azure Databricks 中的诊断日志记录

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../release-notes/release-types.md)提供。

Azure Databricks 提供全面的 Azure Databricks 用户所执行活动的端到端诊断日志，使企业能够监视详细的 Azure Databricks 使用模式。

## <a name="configure-diagnostic-log-delivery"></a>配置诊断日志传送

> [!NOTE]
>
> 诊断日志需要 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)。

1. 以 Azure Databricks 工作区的所有者或参与者身份登录到 Azure 门户，然后单击你的 Azure Databricks 服务资源。
2. 在侧栏的“监视”部分中，单击“诊断设置”选项卡。
3. 单击“启用诊断”。

   > [!div class="mx-imgBorder"]
   > ![Azure Databricks 诊断设置](../../_static/images/audit-logs/azure-diagnostic-settings.png)

4. 在“诊断设置”页上，提供以下配置：

   **名称**

   为要创建的日志输入名称。

   **存档到存储帐户**

   要使用此选项，需要一个可连接到的现有存储帐户。 要在门户中创建新的存储帐户，请参阅[创建存储帐户](/storage/common/storage-create-storage-account)，并按照说明创建 Azure 资源管理器（即通用帐户）。 然后在门户中返回到此页，选择存储帐户。 新创建的存储帐户可能几分钟后才会显示在下拉菜单中。 若要了解写入到存储帐户引发的额外成本，请参阅 [Azure 存储定价](https://azure.microsoft.com/pricing/details/storage/)。

   **流式传输到事件中心**

   若要使用此选项，你需要一个可连接到的现有 Azure 事件中心命名空间和事件中心。 要创建事件中心命名空间，请参阅[使用 Azure 门户创建事件中心命名空间和事件中心](/event-hubs/event-hubs-create)。 然后，在门户中返回到此页，选择事件中心命名空间和策略名称。 若要了解写入到事件中心产生的额外成本，请参阅 [Azure 事件中心定价](https://azure.microsoft.com/pricing/details/event-hubs/)。

   **发送到 Log Analytics**

   若要使用此选项，请使用现有的 Log Analytics 工作区，或者创建新的工作区，只需在门户中执行[创建新工作区](/azure-monitor/learn/quick-collect-azurevm#create-a-workspace)所需的步骤即可。 若要了解向 Log Analytics 发送日志产生的额外成本，请参阅 [Azure Monitor 定价](https://azure.microsoft.com/pricing/details/monitor/)。

   > [!div class="mx-imgBorder"]
   > ![Azure Databricks 诊断设置](../../_static/images/audit-logs/azure-diagnostic-settings2.png)

5. 选择你需要其诊断日志的服务，并设置保留策略。

   保留仅应用于存储帐户。 如果不想应用保留策略，而是想永久保留数据，请将“保留期(天)”设置为 0。

6. 选择“保存” 。
7. 如果收到一个错误，指出“无法更新 <workspace name> 的诊断。 订阅 <subscription id> 未注册为使用 microsoft.insights”，请遵照[排查 Azure 诊断问题](/log-analytics/log-analytics-azure-storage)中的说明注册帐户，然后重试此过程。
8. 若要更改在将来的任意时间点保存诊断日志的方式，可以返回到此页，修改帐户的诊断日志设置。

## <a name="turn-on-logging-using-powershell"></a>使用 PowerShell 启用日志记录

1. 启动 Azure PowerShell 会话，并使用以下命令登录用户的 Azure 帐户：

   ```powershell
   Connect-AzAccount
   ```

   如果尚未安装 Azure Powershell，请使用以下命令安装 Azure PowerShell 并导入 Azure RM 模块。

   ```powershell
   Install-Module -Name Az -AllowClobber
   Import-Module AzureRM
   ```

2. 在弹出的浏览器窗口中，输入 Azure 帐户用户名和密码。 Azure PowerShell 会获取与此帐户关联的所有订阅，并按默认使用第一个订阅。

   如果有多个订阅，可能需要指定用来创建 Azure Key Vault 的特定订阅。 若要查看帐户的订阅，请键入以下命令：

   ```powershell
   Get-AzSubscription
   ```

   若要指定与要记录的 Azure Databricks 帐户关联的订阅，请键入以下命令：

   ```powershell
   Set-AzContext -SubscriptionId <subscription ID>
   ```

3. 将你的 Log Analyticss 资源名称设置为名为 `logAnalytics` 的变量，其中 `ResourceName` 是 Log Analytics 工作区的名称。

   ```powershell
   $logAnalytics = Get-AzResource -ResourceGroupName <resource group name> -ResourceName <resource name> -ResourceType "Microsoft.OperationalInsights/workspaces"
   ```

4. 将 Azure Databricks 服务资源名称设置为名为 `databricks` 的变量，其中 `ResourceName` 是 Azure Databricks 服务的名称。

   ```powershell
   $databricks = Get-AzResource -ResourceGroupName <your resource group name> -ResourceName <your Azure Databricks service name> -ResourceType "Microsoft.Databricks/workspaces"
   ```

5. 若要为 Azure Databricks 启用日志记录，请使用 **Set-AzDiagnosticSetting** cmdlet，同时使用新存储帐户、Azure Databricks 服务和要为其启用日志记录的类别的变量。 运行以下命令，并将 `-Enabled` 标志设置为 `$true`：

   ```powershell
   Set-AzDiagnosticSetting -ResourceId $databricks.ResourceId -WorkspaceId $logAnalytics.ResourceId -Enabled $true -name "<diagnostic setting name>" -Category <comma separated list>
   ```

## <a name="enable-logging-by-using-azure-cli"></a>使用 Azure CLI 启用日志记录

1. 打开 PowerShell。
2. 使用以下命令连接到你的 Azure 帐户：

   ```powershell
   az login
   ```

3. 运行以下诊断设置命令：

   ```powershell
   az monitor diagnostic-settings create --name <diagnostic name>
   --resource-group <log analytics workspace resource group>
   --workspace <log analytics name or object ID>
   --resource <target resource object ID>
   --logs '[
   {
     \"category\": <category name>,
     \"enabled\": true
   }
   ]'
   ```

## <a name="rest-api"></a>REST API

使用 [LogSettings](https://docs.microsoft.com/rest/api/monitor/diagnosticsettings/createorupdate#logsettings) API。

### <a name="request"></a>请求

```
PUT https://management.azure.com/{resourceUri}/providers/microsoft.insights/diagnosticSettings/{name}?api-version=2017-05-01-preview
```

### <a name="request-body"></a>请求正文

```json
{
    "properties": {
    "workspaceId": "<log analytics resourceId>",
    "logs": [
      {
        "category": "<category name>",
        "enabled": true,
        "retentionPolicy": {
          "enabled": false,
          "days": 0
        }
      }
    ]
  }
}
```

## <a name="diagnostic-log-delivery"></a>诊断日志传送

为帐户启用日志记录后，Azure Databricks 会自动开始定期将诊断日志发送到你的传送位置。 日志会在激活后的 24 到 72 小时内可用。 在任何给定的日期，Azure Databricks 都将在前 24 小时内提供至少 99% 的诊断日志，剩余的 1% 将在 72 小时内提供。

## <a name="diagnostic-log-schema"></a>诊断日志架构

诊断日志记录的架构如下所示：

| 字段                                   | 说明                                                                                                                                                                                                                            |
|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **operationversion**                    | 诊断日志格式的架构版本。                                                                                                                                                                                       |
| **time**                                | 操作的 UTC 时间戳。                                                                                                                                                                                                           |
| **properties.sourceIPAddress**          | 源请求的 IP 地址。                                                                                                                                                                                                  |
| **properties.userAgent**                | 用于发出请求的浏览器或 API 客户端。                                                                                                                                                                                    |
| **properties.sessionId**                | 操作的会话 ID。                                                                                                                                                                                                              |
| **identities**                          | 有关发出请求的用户的信息：<br><br>* **email** ：用户电子邮件地址。                                                                                                                                            |
| **category**                            | 记录了请求的服务。                                                                                                                                                                                                   |
| **operationName**                       | 操作，例如登录、注销、读取、写入，等等。                                                                                                                                                                                   |
| **properties.requestId**                | 唯一的请求 ID。                                                                                                                                                                                                                     |
| **properties.requestParams**            | 事件中使用的参数键值对。                                                                                                                                                                                           |
| **properties.response**                 | 对请求的响应：<br><br>* **errorMessage** ：错误消息（如果有错误）。<br>* **result** ：请求的结果。<br>* **statusCode** ：指示请求是否成功的 HTTP 状态代码。 |
| **properties.logId**                    | 日志消息的唯一标识符。                                                                                                                                                                                            |

## <a name="events"></a>事件

`category` 和 `operationName` 属性标识日志记录中的事件。 Azure Databricks 提供以下服务的诊断日志：

* DBFS
* 群集
* 池
* 帐户
* 作业
* 笔记本
* SSH
* 工作区
* 机密
* SQL 权限

如果操作花费很长时间，则会单独记录请求和响应，但请求和响应对具有相同的 `properties.requestId`。

除了与装载相关的操作之外，Azure Databricks 诊断日志不包含与 DBFS 相关的操作。

> [!NOTE]
>
> 自动化操作（例如，由于自动缩放而重设群集大小，或由于调度而启动作业）由用户 _System-User_ 执行。

## <a name="sample-log-output"></a>示例日志输出

下面的 JSON 示例是 Azure Databricks 日志输出的示例：

```json
{
    "TenantId": "<your tenant id",
    "SourceSystem": "|Databricks|",
    "TimeGenerated": "2019-05-01T00:18:58Z",
    "ResourceId": "/SUBSCRIPTIONS/SUBSCRIPTION_ID/RESOURCEGROUPS/RESOURCE_GROUP/PROVIDERS/MICROSOFT.DATABRICKS/WORKSPACES/PAID-VNET-ADB-PORTAL",
    "OperationName": "Microsoft.Databricks/jobs/create",
    "OperationVersion": "1.0.0",
    "Category": "jobs",
    "Identity": {
        "email": "mail@contoso.com",
        "subjectName": null
    },
    "SourceIPAddress": "131.0.0.0",
    "LogId": "201b6d83-396a-4f3c-9dee-65c971ddeb2b",
    "ServiceName": "jobs",
    "UserAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.108 Safari/537.36",
    "SessionId": "webapp-cons-webapp-01exaj6u94682b1an89u7g166c",
    "ActionName": "create",
    "RequestId": "ServiceMain-206b2474f0620002",
    "Response": {
        "statusCode": 200,
        "result": "{\"job_id\":1}"
    },
    "RequestParams": {
        "name": "Untitled",
        "new_cluster": "{\"node_type_id\":\"Standard_DS3_v2\",\"spark_version\":\"5.2.x-scala2.11\",\"num_workers\":8,\"spark_conf\":{\"spark.databricks.delta.preview.enabled\":\"true\"},\"cluster_creator\":\"JOB_LAUNCHER\",\"spark_env_vars\":{\"PYSPARK_PYTHON\":\"/databricks/python3/bin/python3\"},\"enable_elastic_disk\":true}"
    },
    "Type": "DatabricksJobs"
}
```

## <a name="analyze-diagnostic-logs"></a>分析诊断日志

如果在启用诊断日志记录时选择了“发送到 Log Analytics”选项，则容器中的诊断数据会在 24 到 72 小时内转发到 Azure Monitor 日志。

在查看日志之前，请验证 **Log Analytics** 工作区是否已升级为使用新的 Kusto 查询语言。 若要检查，请打开 [Azure 门户](https://portal.azure.com/)并选择最左侧的“Log Analytics”。 然后选择你的 Log Analytics 工作区。 如果收到要升级的消息，请参阅[将 Azure Log Analytics 工作区升级到新的日志搜索](/log-analytics/log-analytics-log-search-upgrade)。

若要查看 Azure Monitor 日志中的诊断数据，请打开左侧菜单中的“日志搜索”页或该页的“管理”区域。 然后，将查询输入到“日志搜索”框中。

> [!div class="mx-imgBorder"]
> ![Azure Log Analytics](../../_static/images/audit-logs/azure-log-analytics.png)

### <a name="queries"></a>查询

可在“日志搜索”框中输入下面这些附加的查询。 这些查询以 Kusto 查询语言编写。

* 若要查询已访问 Azure Databricks 工作区及其位置的所有用户，请执行以下语句：

  ```sql
  DatabricksAccounts
  | where ActionName contains "login"
  | extend d=parse_json(Identity)
  | project UserEmail=d.email, SourceIPAddress
  ```

* 若要检查使用的 Spark 版本，请执行以下语句：

  ```sql
  DatabricksClusters
  | where ActionName == "create"
  | extend d=parse_json(RequestParams)
  | extend SparkVersion= d.spark_version
  | summarize Count=count() by tostring(SparkVersion)
  ```