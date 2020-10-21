---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: 作业 API - Azure Databricks
description: 了解 Databricks 作业 API。
ms.openlocfilehash: fffcaa4cda5f1206edb376682e2120ff55b2fd54
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937681"
---
# <a name="jobs-api"></a>作业 API

利用作业 API，可以创建、编辑和删除作业。
对作业 API 的请求的最大允许大小为 10MB。 有关此 API 的操作指南，请参阅[作业 API 示例](examples.md#jobs-api-example)。

> [!NOTE]
>
> 在发出作业 API 请求时，如果收到 500 级错误，建议最多重试请求 10 分钟（两次重试之间至少间隔 30 秒）。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="create"></a><a id="create"> </a><a id="jobsjobsservicecreatejob"> </a>创建

| 端点                | HTTP 方法     |
|-------------------------|-----------------|
| `2.0/jobs/create`       | `POST`          |

创建新作业。

下面是一个作业的示例请求，该作业每天晚上 10:15 运行：

```json
{
  "name": "Nightly model training",
  "new_cluster": {
    "spark_version": "5.3.x-scala2.11",
    "node_type_id": "Standard_D3_v2",
    "num_workers": 10
  },
  "libraries": [
    {
      "jar": "dbfs:/my-jar.jar"
    },
    {
      "maven": {
        "coordinates": "org.jsoup:jsoup:1.7.2"
      }
    }
  ],
  "timeout_seconds": 3600,
  "max_retries": 1,
  "schedule": {
    "quartz_cron_expression": "0 15 22 ? * *",
    "timezone_id": "America/Los_Angeles"
  },
  "spark_jar_task": {
    "main_class_name": "com.databricks.ComputeModels"
  }
}
```

以及响应：

```json
{
  "job_id": 1
}
```

### <a name="request-structure"></a><a id="jobscreatejob"> </a><a id="request-structure"> </a>请求结构

创建新作业。

> [!IMPORTANT]
>
> * 在新作业群集上运行作业时，该作业会被视为遵从作业计算定价标准的作业计算（自动化）工作负载。
> * 在现有的通用群集上运行作业时，该作业会被视为遵从通用计算定价标准的通用计算（交互式）工作负载。

| 字段名称                                                                | 类型                                                                                                                                                         | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| existing_cluster_id 或 new_cluster                                        | `STRING` 或 [NewCluster](#jobsclusterspecnewcluster)                                                                                                         | 如果是 existing_cluster_id，则该项是将用于此作业的所有运行的现有群集的 ID。 在现有群集上运行作业时，如果该群集停止响应，则可能需要手动重启该群集。 建议在新群集上运行作业，以获得更高的可靠性。<br><br>如果是 new_cluster，则该项是将为每个运行创建的群集的说明。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| notebook_task 或 spark_jar_task 或 spark_python_task 或 spark_submit_task | [NotebookTask](#jobsnotebooktask) 或 [SparkJarTask](#jobssparkjartask) 或 [SparkPythonTask](#jobssparkpythontask) 或 [SparkSubmitTask](#jobssparksubmittask) | 如果是 notebook_task，则表明此作业应该运行笔记本。 此字段不能与 spark_jar_task 一起指定。<br><br>如果是 spark_jar_task，则表明此作业应该运行 JAR。<br><br>如果是 spark_python_task，则表明此作业应该运行 Python 文件。<br><br>如果是 spark_submit_task，则表明此作业应该运行 spark-submit 脚本。                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| name                                                                      | `STRING`                                                                                                                                                     | 可选的作业名称。 默认值是 `Untitled`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 库                                                                 | 一个由[库](libraries.md#managedlibrarieslibrary)构成的数组                                                                                                  | 可选的库列表，这些库会安装在将要执行该作业的群集上。 默认值为空列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| email_notifications                                                       | [JobEmailNotifications](#jobsjobsettingsjobemailnotifications)                                                                                               | 一组可选的电子邮件地址，在此作业的运行开始和完成时，以及在删除此作业时，这些地址会收到通知。 默认行为是不发送任何电子邮件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| timeout_seconds                                                           | `INT32`                                                                                                                                                      | 可选的超时设置，应用于此作业的每个运行。 默认行为是没有任何超时。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| max_retries                                                               | `INT32`                                                                                                                                                      | 对未成功的运行进行重试的最大次数，可选。 如果运行在完成时其以下状态为 `FAILED`，则会被视为未成功：result_state 或<br>`INTERNAL_ERROR`<br>`life_cycle_state`. 值 -1 表示要无限次重试，而值 0 则表示从不重试。 默认行为是从不重试。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| min_retry_interval_millis                                                 | `INT32`                                                                                                                                                      | 可选的最小间隔（失败运行的开始时间与随后的重试运行开始时间之间的间隔），以毫秒为单位。 默认行为是立即重试未成功的运行。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| retry_on_timeout                                                          | `BOOL`                                                                                                                                                       | 一个可选的策略，用于指定在作业超时是否重试该作业。默认行为是在超时发生时不重试。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| schedule                                                                  | [CronSchedule](#jobscronschedule)                                                                                                                            | 此作业的可选定期计划。 默认行为是，在通过单击作业 UI 中的“立即运行”或向 `runNow` 发送 API 请求来触发作业时，该作业会运行。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| max_concurrent_runs                                                       | `INT32`                                                                                                                                                      | 允许的作业最大并发运行数，可选。<br><br>如果希望能够以并发方式执行同一作业的多个运行，请设置此值。 设置此值适用于这样的情形：例如，如果你按计划频繁触发作业并希望允许连续的运行彼此重叠，或者，如果你希望触发多个在输入参数方面有区别的运行。<br><br>此设置只影响新的运行。 例如，假定作业的并发数为 4，并且存在 4 个并发的活动运行。 那么，将该并发数设置为 3 则不会终止任何活动的运行。 但是，从此之后，除非活动的运行少于 3 个，否则会跳过新的运行。<br><br>此值不能超过 1000。 将此值设置为 0 会导致跳过所有新的运行。 默认行为是只允许 1 个并发运行。 |

### <a name="response-structure"></a><a id="jobscreatejobresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型          | 描述                                             |
|----------------|---------------|---------------------------------------------------------|
| job_id         | `INT64`       | 新创建的作业的规范标识符。     |

## <a name="list"></a><a id="jobsjobsservicelistjobs"> </a><a id="list"> </a>列表

| 端点              | HTTP 方法     |
|-----------------------|-----------------|
| `2.0/jobs/list`       | `GET`           |

列出所有作业。
示例响应：

```json
{
  "jobs": [
    {
      "job_id": 1,
      "settings": {
        "name": "Nightly model training",
        "new_cluster": {
          "spark_version": "5.3.x-scala2.11",
          "node_type_id": "Standard_D3_v2",
          "num_workers": 10
        },
        "libraries": [
          {
            "jar": "dbfs:/my-jar.jar"
          },
          {
            "maven": {
              "coordinates": "org.jsoup:jsoup:1.7.2"
            }
          }
        ],
        "timeout_seconds": 100000000,
        "max_retries": 1,
        "schedule": {
          "quartz_cron_expression": "0 15 22 ? * *",
          "timezone_id": "America/Los_Angeles",
          "pause_status": "UNPAUSED"
        },
        "spark_jar_task": {
          "main_class_name": "com.databricks.ComputeModels"
        }
      },
      "created_time": 1457570074236
    }
  ]
}
```

### <a name="response-structure"></a><a id="jobslistjobsresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型                           | 描述           |
|----------------|--------------------------------|-----------------------|
| jobs           | 一个由[作业](#jobsjob)构成的数组    | 作业的列表。     |

## <a name="delete"></a><a id="delete"> </a><a id="jobsjobsservicedeletejob"> </a>删除

| 端点                | HTTP 方法     |
|-------------------------|-----------------|
| `2.0/jobs/delete`       | `POST`          |

删除该作业并向 `JobSettings.email_notifications` 中指定的地址发送电子邮件。
如果已删除该作业，则不会执行任何操作。 在该作业被删除之后，通过作业 UI 或 API 均无法查看其详细信息或其运行历史记录。 在此请求完成时，该作业一定会被删除。 但是，在接收此请求之前处于活动状态的运行可能仍会处于活动状态。 它们将会被异步终止。

示例请求：

```json
{
  "job_id": 1
}
```

### <a name="request-structure"></a><a id="jobsdeletejob"> </a><a id="request-structure"> </a>请求结构

删除某个作业并向 `JobSettings.email_notifications` 中指定的地址发送电子邮件。
如果已删除该作业，则不会执行任何操作。 在该作业被删除之后，通过作业 UI 或 API 均无法查看其详细信息或其运行历史记录。 在此请求完成时，该作业一定会被删除。 但是，在接收此请求之前处于活动状态的运行可能仍会处于活动状态。 它们将会被异步终止。

| 字段名称     | 类型          | 描述                                                            |
|----------------|---------------|------------------------------------------------------------------------|
| job_id         | `INT64`       | 要删除的作业的规范标识符。 此字段为必需字段。 |

## <a name="get"></a><a id="get"> </a><a id="jobsjobsservicegetjob"> </a>获取

| 端点             | HTTP 方法     |
|----------------------|-----------------|
| `2.0/jobs/get`       | `GET`           |

检索有关某一个作业的信息。
示例请求：

```bash
/jobs/get?job_id=1
```

示例响应：

```json
{
  "job_id": 1,
  "settings": {
    "name": "Nightly model training",
    "new_cluster": {
      "spark_version": "5.3.x-scala2.11",
      "node_type_id": "Standard_D3_v2",
      "num_workers": 10
    },
    "libraries": [
      {
        "jar": "dbfs:/my-jar.jar"
      },
      {
        "maven": {
          "coordinates": "org.jsoup:jsoup:1.7.2"
        }
      }
    ],
    "timeout_seconds": 100000000,
    "max_retries": 1,
    "schedule": {
      "quartz_cron_expression": "0 15 22 ? * *",
      "timezone_id": "America/Los_Angeles",
      "pause_status": "UNPAUSED"
    },
    "spark_jar_task": {
      "main_class_name": "com.databricks.ComputeModels"
    }
  },
  "created_time": 1457570074236
}
```

### <a name="request-structure"></a><a id="jobsgetjob"> </a><a id="request-structure"> </a>请求结构

检索有关某一个作业的信息。

| 字段名称     | 类型          | 描述                                                                                |
|----------------|---------------|--------------------------------------------------------------------------------------------|
| job_id         | `INT64`       | 要检索其相关信息的作业的规范标识符。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="jobsgetjobresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称            | 类型                            | 描述                                                                                               |
|-----------------------|---------------------------------|-----------------------------------------------------------------------------------------------------------|
| job_id                | `INT64`                         | 此作业的规范标识符。                                                                    |
| creator_user_name     | `STRING`                        | 创建者用户名。 如果已将该用户删除，响应中将不会包含此字段。 |
| 设置              | [JobSettings](#jobsjobsettings) | 此作业及其所有运行的设置。 可以使用 `resetJob` 方法来更新这些设置。     |
| created_time          | `INT64`                         | 此作业的创建时间，以 epoch 毫秒表示（自 UTC 1970 年 1 月 1 日起的毫秒数）。           |

## <a name="reset"></a><a id="jobsjobsserviceresetjob"> </a><a id="reset"> </a>重置

| 端点               | HTTP 方法     |
|------------------------|-----------------|
| `2.0/jobs/reset`       | `POST`          |

覆盖作业设置。

使作业 2 类似于作业 1（来自 `create_job` 示例）的示例请求：

```json
{
  "job_id": 2,
  "new_settings": {
    "name": "Nightly model training",
    "new_cluster": {
      "spark_version": "5.3.x-scala2.11",
      "node_type_id": "Standard_D3_v2",
      "num_workers": 10
    },
    "libraries": [
      {
        "jar": "dbfs:/my-jar.jar"
      },
      {
        "maven": {
          "coordinates": "org.jsoup:jsoup:1.7.2"
        }
      }
    ],
    "timeout_seconds": 100000000,
    "max_retries": 1,
    "schedule": {
      "quartz_cron_expression": "0 15 22 ? * *",
      "timezone_id": "America/Los_Angeles",
      "pause_status": "UNPAUSED"
    },
    "spark_jar_task": {
      "main_class_name": "com.databricks.ComputeModels"
    }
  }
}
```

### <a name="request-structure"></a><a id="jobsresetjob"> </a><a id="request-structure"> </a>请求结构

覆盖作业设置。

| 字段名称       | 类型                            | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                |
|------------------|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| job_id           | `INT64`                         | 要重置的作业的规范标识符。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                      |
| new_settings     | [JobSettings](#jobsjobsettings) | 作业的新设置。 这些新设置会完全替换旧设置。<br><br>对以下字段的更改不会应用于活动的运行：<br>`JobSettings.cluster_spec` 或 `JobSettings.task`。<br><br>对以下字段的更改会应用于活动的运行以及将来的运行：<br>`JobSettings.timeout_second`、`JobSettings.email_notifications` 或 <br>`JobSettings.retry_policy`. 此字段为必需字段。 |

## <a name="run-now"></a><a id="jobsjobsservicerunnow"> </a><a id="run-now"> </a>立即运行

> [!IMPORTANT]
>
> * 作业数限制为 1000。
> * 工作区在一小时内可以创建的作业数限制为 5000（包括“立即运行”和“运行提交”）。 此限制还会影响 REST API 和笔记本工作流创建的作业。
> * 工作区限制为 150 个并发（正在运行的）作业运行。
> * 工作区限制为 1000 个活动（正在运行和挂起的）作业运行。

| 端点                 | HTTP 方法     |
|--------------------------|-----------------|
| `2.0/jobs/run-now`       | `POST`          |

立即运行作业，并返回已触发的运行的 run_id。

> [!NOTE]
>
> 如果你发现自己经常将“[创建](#jobsjobsservicecreatejob)”与“[立即运行](#jobsjobsservicerunnow)”一起使用，那么你可以考虑使用[运行提交](#jobsjobsservicesubmitrun) API。 利用此 API 终结点可以直接提交工作负载，而无需在 Azure Databricks 中创建作业。

笔记本作业的示例请求：

```json
{
  "job_id": 1,
  "notebook_params": {
    "dry-run": "true",
    "oldest-time-to-consider": "1457570074236"
  }
}
```

JAR 作业的示例请求：

```json
{
  "job_id": 2,
  "jar_params": ["param1", "param2"]
}
```

### <a name="request-structure"></a><a id="jobsrunnow"> </a><a id="request-structure"> </a>请求结构

立即运行作业，并返回已触发的运行的 run_id。

| 字段名称              | 类型                                 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------------------|--------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| job_id                  | `INT64`                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| jar_params              | 一个由 `STRING` 构成的数组                 | 具有 JAR 任务的作业的参数列表，例如 `"jar_params": ["john doe", "35"]`。 这些参数将用于调用 Spark JAR 任务中指定的 main 类的 main 函数。 如果未在调用 `run-now` 时指定，该项将默认为空列表。 jar_params 不能与 notebook_params 一起指定。 此字段的 JSON 表示形式（即 `{"jar_params":["john doe","35"]}`）不能超过 10,000 字节。                                                                                                                            |
| notebook_params         | [ParamPair](#jobsparampair) 的映射 | 具有笔记本任务的作业的从键到值的映射，例如<br>`"notebook_params": {"name": "john doe", "age":  "35"}`. 该映射会传递到笔记本，并且可通过 [dbutils.widgets.get](../../databricks-utils.md#dbutils-widgets) 函数访问。<br><br>如果未在调用 `run-now` 时指定，已触发的运行会使用该作业的基参数。<br><br>notebook_params 不能与 jar_params 一起指定。<br><br>此字段的 JSON 表示形式（即<br>`{"notebook_params":{"name":"john doe","age":"35"}}`）不能超过 10,000 字节。 |
| python_params           | 一个由 `STRING` 构成的数组                 | 具有 Python 任务的作业的参数列表，例如 `"python_params": ["john doe", "35"]`。 这些参数将会作为命令行参数传递到 Python 文件。 如果在调用 `run-now` 时指定，则此项会覆盖作业设置中指定的参数。 此字段的 JSON 表示形式（即 `{"python_params":["john doe","35"]}`）不能超过 10,000 字节。                                                                                                                                                                                                   |
| spark_submit_params     | 一个由 `STRING` 构成的数组                 | 具有“Spark 提交”任务的作业的参数列表，例如<br>`"spark_submit_params": ["--class", "org.apache.spark.examples.SparkPi"]`. 这些参数将会作为命令行参数传递到 spark-submit 脚本。 如果在调用 `run-now` 时指定，则此项会覆盖作业设置中指定的参数。 此字段的 JSON 表示形式不能超过 10,000 字节。                                                                                                                                                                                            |

### <a name="response-structure"></a><a id="jobsrunnowresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称        | 类型          | 描述                                                    |
|-------------------|---------------|----------------------------------------------------------------|
| run_id            | `INT64`       | 新触发的运行的全局唯一 ID。             |
| number_in_job     | `INT64`       | 此运行在该作业的所有运行中的序列号。     |

## <a name="runs-submit"></a><a id="jobsjobsservicesubmitrun"> </a><a id="runs-submit"> </a>运行提交

> [!IMPORTANT]
>
> * 作业数限制为 1000。
> * 工作区在一小时内可以创建的作业数限制为 5000（包括“立即运行”和“运行提交”）。 此限制还会影响 REST API 和笔记本工作流创建的作业。
> * 工作区限制为 150 个并发（正在运行的）作业运行。
> * 工作区限制为 1000 个活动（正在运行和挂起的）作业运行。

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/jobs/runs/submit`       | `POST`          |

提交一次性运行。 此终结点不要求创建 Databricks 作业。  你可以直接提交工作负载。 通过此终结点提交的运行不会显示在 UI 中。 提交运行之后，请使用 `jobs/runs/get` API 来检查运行状态。

示例请求：

```json
{
  "run_name": "my spark task",
  "new_cluster": {
    "spark_version": "5.3.x-scala2.11",
    "node_type_id": "Standard_D3_v2",
    "num_workers": 10
  },
  "libraries": [
    {
      "jar": "dbfs:/my-jar.jar"
    },
    {
      "maven": {
        "coordinates": "org.jsoup:jsoup:1.7.2"
      }
    }
  ],
  "spark_jar_task": {
    "main_class_name": "com.databricks.ComputeModels"
  }
}
```

以及响应：

```json
{
  "run_id": 123
}
```

### <a name="request-structure"></a><a id="jobssubmitrun"> </a><a id="request-structure"> </a>请求结构

使用提供的设置提交新运行。

> [!IMPORTANT]
>
> * 在新作业群集上运行作业时，该作业会被视为遵从作业计算定价标准的作业计算（自动化）工作负载。
> * 在现有的通用群集上运行作业时，该作业会被视为遵从通用计算定价标准的通用计算（交互式）工作负载。

| 字段名称                                                                | 类型                                                                                                                                                         | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| existing_cluster_id 或 new_cluster                                        | `STRING` 或 [NewCluster](#jobsclusterspecnewcluster)                                                                                                         | 如果是 existing_cluster_id，则该项是将用于此作业的所有运行的现有群集的 ID。 在现有群集上运行作业时，如果该群集停止响应，则可能需要手动重启该群集。 建议在新群集上运行作业，以获得更高的可靠性。<br><br>如果是 new_cluster，则该项是将为每个运行创建的群集的说明。                                                                                                                           |
| notebook_task 或 spark_jar_task 或 spark_python_task 或 spark_submit_task | [NotebookTask](#jobsnotebooktask) 或 [SparkJarTask](#jobssparkjartask) 或 [SparkPythonTask](#jobssparkpythontask) 或 [SparkSubmitTask](#jobssparksubmittask) | 如果是 notebook_task，则表明此作业应该运行笔记本。 此字段不能与 spark_jar_task 一起指定。<br><br>如果是 spark_jar_task，则表明此作业应该运行 JAR。<br><br>如果是 spark_python_task，则表明此作业应该运行 Python 文件。<br><br>如果是 spark_submit_task，则表明此作业应该运行 spark-submit 脚本。                                                                                                                     |
| run_name                                                                  | `STRING`                                                                                                                                                     | 可选的作业名称。 默认值是 `Untitled`。                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| 库                                                                 | 一个由[库](libraries.md#managedlibrarieslibrary)构成的数组                                                                                                  | 可选的库列表，这些库会安装在将要执行该作业的群集上。 默认值为空列表。                                                                                                                                                                                                                                                                                                                                                                      |
| timeout_seconds                                                           | `INT32`                                                                                                                                                      | 可选的超时设置，应用于此作业的每个运行。 默认行为是没有任何超时。                                                                                                                                                                                                                                                                                                                                                                                                 |
| idempotency_token                                                         | `STRING`                                                                                                                                                     | 可选令牌，可用于保证作业运行请求的幂等性。 如果已经存在具有提供的令牌的活动运行，该请求将不会创建新运行，而是会返回现有运行的 ID。<br><br>如果指定幂等性令牌，则可在失败时重试，直到该请求成功。 Azure Databricks 会确保只有一个运行将通过该幂等性令牌启动。<br><br>此令牌最多只能包含 64 个字符。 |

### <a name="response-structure"></a><a id="jobssubmitrunresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型          | 描述                                               |
|----------------|---------------|-----------------------------------------------------------|
| run_id         | `INT64`       | 新提交的运行的规范标识符。     |

## <a name="runs-list"></a><a id="jobsjobsservicelistruns"> </a><a id="runs-list"> </a>运行列表

| 端点                   | HTTP 方法     |
|----------------------------|-----------------|
| `2.0/jobs/runs/list`       | `GET`           |

按启动时间（从近到远）列出运行。

> [!NOTE]
>
> 运行在 60 天之后会自动删除。 如果在 60 天之后还需要引用这些运行，则应在它们过期之前保存旧运行结果。 若要使用 UI 导出，请参阅[导出作业运行结果](../../../jobs.md#export-job-runs)。 若要使用作业 API 导出，请参阅[运行导出](#jobsjobsserviceexportrun)。

示例请求：

```bash
/jobs/runs/list?job_id=1&active_only=false&offset=1&limit=1
```

以及响应：

```json
{
  "runs": [
    {
      "job_id": 1,
      "run_id": 452,
      "number_in_job": 5,
      "state": {
        "life_cycle_state": "RUNNING",
        "state_message": "Performing action"
      },
      "task": {
        "notebook_task": {
          "notebook_path": "/Users/donald@duck.com/my-notebook"
        }
      },
      "cluster_spec": {
        "existing_cluster_id": "1201-my-cluster"
      },
      "cluster_instance": {
        "cluster_id": "1201-my-cluster",
        "spark_context_id": "1102398-spark-context-id"
      },
      "overriding_parameters": {
        "jar_params": ["param1", "param2"]
      },
      "start_time": 1457570074236,
      "setup_duration": 259754,
      "execution_duration": 3589020,
      "cleanup_duration": 31038,
      "trigger": "PERIODIC"
    }
  ],
  "has_more": true
}
```

### <a name="request-structure"></a><a id="jobslistruns"> </a><a id="request-structure"> </a>请求结构

按启动时间（从近到远）列出运行。

| 字段名称                                | 类型                     | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|-------------------------------------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| active_only 或 completed_only             | `BOOL` 或 `BOOL`         | 如果 active_only 为 `true`，则结果中只包括活动运行；否则会将活动运行和已完成的运行都列出。 活动运行是 [RunLifecycleState](#jobsrunlifecyclestate) 为 `PENDING`、`RUNNING` 或 `TERMINATING` 的运行。 在 completed_only 为 `true` 时，此字段不能为 `true`。<br><br>如果 completed_only 为 `true`，则结果中仅包括已完成的运行；否则会将活动运行和已完成的运行都列出。 在 active_only 为 `true` 时，此字段不能为 `true`。 |
| job_id                                    | `INT64`                  | 要列出其运行的作业。 如果省略，作业服务将会列出所有作业中的运行。                                                                                                                                                                                                                                                                                                                                                                                                          |
| offset                                    | `INT32`                  | 要返回的第一个运行的偏移量（相对于最近的运行）。                                                                                                                                                                                                                                                                                                                                                                                                                             |
| limit                                     | `INT32`                  | 要返回的运行数。 此值应大于 0 且小于 1000。 默认值为 20。 如果请求将限制指定为 0，该服务将会改用最大限制。                                                                                                                                                                                                                                                                                                 |

### <a name="response-structure"></a><a id="jobslistrunsresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型                           | 描述                                                                          |
|----------------|--------------------------------|--------------------------------------------------------------------------------------|
| 运行           | 一个由[运行](#jobsrun)构成的数组    | 运行的列表，按启动时间（由近到远）列出。                                 |
| has_more       | `BOOL`                         | 如果为 true，则可以列出与提供的筛选器匹配的其他运行。     |

## <a name="runs-get"></a><a id="jobsjobsservicegetrun"> </a><a id="runs-get"> </a>运行获取

| 端点                  | HTTP 方法     |
|---------------------------|-----------------|
| `2.0/jobs/runs/get`       | `GET`           |

检索运行的元数据。

> [!NOTE]
>
> 运行在 60 天之后会自动删除。 如果在 60 天之后还需要引用这些运行，则应在它们过期之前保存旧运行结果。 若要使用 UI 导出，请参阅[导出作业运行结果](../../../jobs.md#export-job-runs)。 若要使用作业 API 导出，请参阅[运行导出](#jobsjobsserviceexportrun)。

示例请求：

```bash
/jobs/runs/get?run_id=452
```

示例响应：

```json
{
  "job_id": 1,
  "run_id": 452,
  "number_in_job": 5,
  "state": {
    "life_cycle_state": "RUNNING",
    "state_message": "Performing action"
  },
  "task": {
    "notebook_task": {
      "notebook_path": "/Users/donald@duck.com/my-notebook"
    }
  },
  "cluster_spec": {
    "existing_cluster_id": "1201-my-cluster"
  },
  "cluster_instance": {
    "cluster_id": "1201-my-cluster",
    "spark_context_id": "1102398-spark-context-id"
  },
  "overriding_parameters": {
    "jar_params": ["param1", "param2"]
  },
  "start_time": 1457570074236,
  "setup_duration": 259754,
  "execution_duration": 3589020,
  "cleanup_duration": 31038,
  "trigger": "PERIODIC"
}
```

### <a name="request-structure"></a><a id="jobsgetrun"> </a><a id="request-structure"> </a>请求结构

检索无任何输出的运行的元数据。

| 字段名称     | 类型          | 描述                                                                                         |
|----------------|---------------|-----------------------------------------------------------------------------------------------------|
| run_id         | `INT64`       | 要检索其元数据的运行的规范标识符。 此字段为必需字段。     |

### <a name="response-structure"></a><a id="jobsgetrunresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称                  | 类型                                    | 描述                                                                                                                                                                                                                                                                      |
|-----------------------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| job_id                      | `INT64`                                 | 包含此运行的作业的规范标识符。                                                                                                                                                                                                                      |
| run_id                      | `INT64`                                 | 该运行的规范标识符。 此 ID 在所有作业的所有运行中都是唯一的。                                                                                                                                                                                              |
| number_in_job               | `INT64`                                 | 此运行在该作业的所有运行中的序列号。 此值从 1 开始。                                                                                                                                                                                               |
| original_attempt_run_id     | `INT64`                                 | 如果此运行是对之前某次运行尝试的重试，则此字段会包含原始尝试的 run_id；否则此字段与本次运行的 run_id 相同。                                                                                                                                  |
| state                       | [RunState](#jobsrunstate)               | 运行的结果和生命周期状态。                                                                                                                                                                                                                                      |
| schedule                    | [CronSchedule](#jobscronschedule)       | 触发此运行的 cron 计划（如果此运行已由定期计划程序触发）。                                                                                                                                                                                         |
| task                        | [JobTask](#jobsjobtask)                 | 由该运行执行的任务（如果有）。                                                                                                                                                                                                                                           |
| cluster_spec                | [ClusterSpec](#jobsclusterspec)         | 创建此运行时作业的群集规范的快照。                                                                                                                                                                                                         |
| cluster_instance            | [ClusterInstance](#jobsclusterinstance) | 用于此运行的群集。 如果将该运行指定为使用新群集，则在作业服务已为该运行请求群集后，将会设置此字段。                                                                                                                   |
| overriding_parameters       | [RunParameters](#jobsrunparameters)     | 用于此运行的参数。                                                                                                                                                                                                                                                |
| start_time                  | `INT64`                                 | 启动此运行的时间，以epoch 毫秒表示（自 UTC 1970 年 1 月 1 日起的毫秒数）。 此时间可能不是作业任务开始执行的时间，例如，如果该作业已计划在新群集上运行，则此时间是发出群集创建调用的时间。 |
| setup_duration              | `INT64`                                 | 设置该群集所花费的时间（以毫秒为单位）。 对于在新群集上运行的运行，此时间是群集创建时间，对于在现有群集上运行的运行，此时间应该很短。                                                                              |
| execution_duration          | `INT64`                                 | 执行 JAR 或笔记本中的命令（直到这些命令已完成、失败、超时、被取消或遇到意外错误）所花费的时间（以毫秒为单位）。                                                                                                     |
| cleanup_duration            | `INT64`                                 | 终止该群集并清理任何中间结果等等所花费的时间（以毫秒为单位）。该运行的总持续时间为 setup_duration、execution_duration 以及 cleanup_duration 之和。                                                          |
| 触发器                     | [TriggerType](#jobstriggertype)         | 触发此运行的触发器的类型，例如，定期计划或一次性运行。                                                                                                                                                                                            |
| creator_user_name           | `STRING`                                | 创建者用户名。 如果已将该用户删除，响应中将不会包含此字段。                                                                                                                                                                        |
| run_page_url                | `STRING`                                | 运行的详细信息页的 URL。                                                                                                                                                                                                                                           |

## <a name="runs-export"></a><a id="jobsjobsserviceexportrun"> </a><a id="runs-export"> </a>运行导出

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/jobs/runs/export`       | `GET`           |

导出并检索作业运行任务。

> [!NOTE]
>
> 只有笔记本运行可以采用 HTML 格式导出。
> 导出其他类型的其他运行将会失败。

示例请求：

```bash
/jobs/runs/export?run_id=452
```

示例响应：

```json
{
  "views": [ {
    "content": "<!DOCTYPE html><html><head>Head</head><body>Body</body></html>",
    "name": "my-notebook",
    "type": "NOTEBOOK"
  } ]
}
```

若要从 JSON 响应中提取 HTML 笔记本，请下载并运行此 [Python 脚本](../../../_static/examples/extract.py)。

> [!NOTE]
>
> `__DATABRICKS_NOTEBOOK_MODEL` 对象中的笔记本正文已编码。

### <a name="request-structure"></a><a id="jobsexportrun"> </a><a id="request-structure"> </a>请求结构

检索作业运行任务的导出。

| 字段名称          | 类型                                | 描述                                                         |
|---------------------|-------------------------------------|---------------------------------------------------------------------|
| run_id              | `INT64`                             | 该运行的规范标识符。 此字段为必需字段。       |
| views_to_export     | [ViewsToExport](#jobsviewstoexport) | 要导出哪些视图（CODE、DASHBOARDS 或 ALL）。 默认为“CODE”。 |

### <a name="response-structure"></a><a id="jobsexportrunresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称     | 类型                                  | 描述                                                         |
|----------------|---------------------------------------|---------------------------------------------------------------------|
| 视图          | 一个由 [ViewItem](#jobsviewitem) 构成的数组 | HTML 格式的导出内容（每个视图项一个）。      |

## <a name="runs-cancel"></a><a id="jobsjobsservicecancelrun"> </a><a id="runs-cancel"> </a>运行取消

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/jobs/runs/cancel`       | `POST`          |

取消运行。 该运行会被异步取消，因此，在此请求完成时，该运行可能仍在运行。 该运行稍后将会被终止。 如果该运行已处于最终的 `life_cycle_state`，则此方法为无操作。

此终结点会验证 `run_id` 参数是否有效，如果参数无效，则会返回 HTTP 状态代码 400。

示例请求：

```json
{
  "run_id": 453
}
```

### <a name="request-structure"></a><a id="jobscancelrun"> </a><a id="request-structure"> </a>请求结构

取消运行。 该运行会被异步取消，因此，在此请求完成时，该运行可能仍处于活动状态。 该运行将会尽快被终止。

| 字段名称     | 类型          | 描述                 |
|----------------|---------------|-----------------------------|
| run_id         | `INT64`       | 此字段为必需字段。     |

## <a name="runs-get-output"></a><a id="jobsjobsservicegetrunoutput"> </a><a id="runs-get-output"> </a>运行获取输出

| 端点                         | HTTP 方法     |
|----------------------------------|-----------------|
| `2.0/jobs/runs/get-output`       | `GET`           |

检索运行的输出。
在笔记本任务通过 [dbutils.notebook.exit()](../../../notebooks/notebook-workflows.md#notebook-workflows-exit) 调用返回一个值时，可以使用此终结点来检索该值。
Azure Databricks 将此 API 限制为返回输出中的前 5 MB。
若要返回更大的结果，可将作业结果存储在云存储服务中。

此终结点会验证 `run_id` 参数是否有效，如果参数无效，则会返回 HTTP 状态代码 400。

运行在 60 天之后会自动删除。 如果在 60 天之后还需要引用这些运行，则应在它们过期之前保存旧运行结果。 若要使用 UI 导出，请参阅[导出作业运行结果](../../../jobs.md#export-job-runs)。
若要使用作业 API 导出，请参阅[运行导出](#jobsjobsserviceexportrun)。

示例请求：

```bash
/jobs/runs/get-output?run_id=453
```

以及响应：

```json
{
  "metadata": {
    "job_id": 1,
    "run_id": 452,
    "number_in_job": 5,
    "state": {
      "life_cycle_state": "TERMINATED",
      "result_state": "SUCCESS",
      "state_message": ""
    },
    "task": {
      "notebook_task": {
        "notebook_path": "/Users/donald@duck.com/my-notebook"
      }
    },
    "cluster_spec": {
      "existing_cluster_id": "1201-my-cluster"
    },
    "cluster_instance": {
      "cluster_id": "1201-my-cluster",
      "spark_context_id": "1102398-spark-context-id"
    },
    "overriding_parameters": {
      "jar_params": ["param1", "param2"]
    },
    "start_time": 1457570074236,
    "setup_duration": 259754,
    "execution_duration": 3589020,
    "cleanup_duration": 31038,
    "trigger": "PERIODIC"
  },
  "notebook_output": {
    "result": "the maybe truncated string passed to dbutils.notebook.exit()"
  }
}
```

### <a name="request-structure"></a><a id="jobsgetrunoutput"> </a><a id="request-structure"> </a>请求结构

检索运行的输出和元数据。

| 字段名称     | 类型          | 描述                                                   |
|----------------|---------------|---------------------------------------------------------------|
| run_id         | `INT64`       | 该运行的规范标识符。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="jobsgetrunoutputresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称                           | 类型                                                          | 描述                                                                                                                                                                                                                                                                                                                                                                                                                               |
|--------------------------------------|---------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| notebook_output 或 error             | [NotebookOutput](#jobsnotebooktasknotebookoutput) 或 `STRING` | 如果是 notebook_output，则该项是笔记本任务的输出（如果可用）。 在未调用的情况下终止（成功或失败）的笔记本任务<br>`dbutils.notebook.exit()` 被视为会有一个空输出。 此字段将会被设置，但其结果值将为空。<br><br>如果是 error，则此项是一个错误消息，指示输出不可用的原因。 该消息是非结构化的，且其确切格式随时可能发生变更。 |
| metadata                             | [运行](#jobsrun)                                               | 该运行的全部详细信息（其输出除外）。                                                                                                                                                                                                                                                                                                                                                                                             |

## <a name="runs-delete"></a><a id="jobsjobsservicedeleterun"> </a><a id="runs-delete"> </a>运行删除

| 端点                     | HTTP 方法     |
|------------------------------|-----------------|
| `2.0/jobs/runs/delete`       | `POST`          |

删除非活动运行。 如果该运行处于活动状态，则返回错误。

示例请求：

```json
{
  "run_id": 42
}
```

### <a name="request-structure"></a><a id="jobsdeleterun"> </a><a id="request-structure"> </a>请求结构

检索无任何输出的运行的元数据。

| 字段名称     | 类型          | 描述                                                                 |
|----------------|---------------|-----------------------------------------------------------------------------|
| run_id         | `INT64`       | 要检索其元数据的运行的规范标识符。     |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="jobadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [ClusterInstance](#clusterinstance)
* [ClusterSpec](#clusterspec)
* [CronSchedule](#cronschedule)
* [作业](#job)
* [JobEmailNotifications](#jobemailnotifications)
* [JobSettings](#jobsettings)
* [JobTask](#jobtask)
* [NewCluster](#newcluster)
* [NotebookOutput](#notebookoutput)
* [NotebookTask](#notebooktask)
* [ParamPair](#parampair)
* [运行](#run)
* [RunParameters](#runparameters)
* [RunState](#runstate)
* [SparkJarTask](#sparkjartask)
* [SparkPythonTask](#sparkpythontask)
* [SparkSubmitTask](#sparksubmittask)
* [ViewItem](#viewitem)
* [RunLifeCycleState](#runlifecyclestate)
* [RunResultState](#runresultstate)
* [TriggerType](#triggertype)
* [ViewType](#viewtype)
* [ViewsToExport](#viewstoexport)

### <a name="clusterinstance"></a><a id="clusterinstance"> </a><a id="jobsclusterinstance"> </a>ClusterInstance

某个运行所使用的群集和 Spark 上下文的标识符。 这两个值总是会共同标识执行上下文。

| 字段名称           | 类型           | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|----------------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cluster_id           | `STRING`       | 某个运行所使用的群集的规范标识符。 此字段始终可用于现有群集上的运行。 对于新群集上的运行，此字段会在创建群集后变为可用。 可使用此值来查看日志，具体方法是浏览到 `/#setting/sparkui/$cluster_id/driver-logs`。 在运行完成之后，日志将继续可用。<br><br>如果此标识符尚不可用，响应将不包含此字段。 |
| spark_context_id     | `STRING`       | 某个运行所使用的 Spark 上下文的规范标识符。 该运行开始执行后，此字段将会被填充。 可使用此值来查看 Spark UI，具体方法是浏览到 `/#setting/sparkui/$cluster_id/$spark_context_id`。 在运行完成之后，Spark UI 将继续可用。<br><br>如果此标识符尚不可用，响应将不包含此字段。                                                   |

### <a name="clusterspec"></a><a id="clusterspec"> </a><a id="jobsclusterspec"> </a>ClusterSpec

> [!IMPORTANT]
>
> * 在新作业群集上运行作业时，该作业会被视为遵从作业计算定价标准的作业计算（自动化）工作负载。
> * 在现有的通用群集上运行作业时，该作业会被视为遵从通用计算定价标准的通用计算（交互式）工作负载。

| 字段名称                                  | 类型                                                                          | 描述                                                                                                                                                                                                                                                                                                                                                            |
|---------------------------------------------|-------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| existing_cluster_id 或 new_cluster          | `STRING` 或 [NewCluster](#jobsclusterspecnewcluster)                          | 如果是 existing_cluster_id，则该项是将用于此作业的所有运行的现有群集的 ID。 在现有群集上运行作业时，如果该群集停止响应，则可能需要手动重启该群集。 建议在新群集上运行作业，以获得更高的可靠性。<br><br>如果是 new_cluster，则该项是将为每个运行创建的群集的说明。 |
| 库                                   | 一个由[库](libraries.md#managedlibrarieslibrary)构成的数组                   | 可选的库列表，这些库会安装在将要执行该作业的群集上。 默认值为空列表。                                                                                                                                                                                                                                            |

### <a name="cronschedule"></a><a id="cronschedule"> </a><a id="jobscronschedule"> </a>CronSchedule

| 字段名称                 | 类型           | 描述                                                                                                                                                                                                                 |
|----------------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| quartz_cron_expression     | `STRING`       | 一个使用 Quartz 语法的 Cron 表达式，用于描述作业的计划。 有关详细信息，请参阅 [Cron 触发器](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/crontrigger.html)。 此字段为必需字段。 |
| timezone_id                | `STRING`       | Java 时区 ID。 将根据此时区解析作业的计划。 有关详细信息，请参阅 [Java 时区](https://docs.oracle.com/javase/7/docs/api/java/util/TimeZone.html)。 此字段为必需字段。      |
| pause_status               | `STRING`       | 指示此计划是否已暂停。 值为“PAUSED”或“UNPAUSED”。                                                                                                                                             |

### <a name="job"></a><a id="job"> </a><a id="jobsjob"> </a>作业

| 字段名称            | 类型                            | 描述                                                                                               |
|-----------------------|---------------------------------|-----------------------------------------------------------------------------------------------------------|
| job_id                | `INT64`                         | 此作业的规范标识符。                                                                    |
| creator_user_name     | `STRING`                        | 创建者用户名。 如果已将该用户删除，响应中将不会包含此字段。 |
| 设置              | [JobSettings](#jobsjobsettings) | 此作业及其所有运行的设置。 可以使用 `resetJob` 方法来更新这些设置。     |
| created_time          | `INT64`                         | 此作业的创建时间，以 epoch 毫秒表示（自 UTC 1970 年 1 月 1 日起的毫秒数）。           |

### <a name="jobemailnotifications"></a><a id="jobemailnotifications"> </a><a id="jobsjobsettingsjobemailnotifications"> </a>JobEmailNotifications

> [!IMPORTANT]
>
> on_start、on_success 和 on_failure 等字段只接受拉丁字符（ASCII 字符集）。 使用非 ASCII 字符将会返回错误。 例如，中文、日文汉字和表情符号都属于无效的非 ASCII 字符。

| 字段名称                         | 类型                       | 描述                                                                                                                                                                                                                                                                                                                                                  |
|------------------------------------|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| on_start                           | 一个由 `STRING` 构成的数组       | 一个电子邮件地址列表，其中的电子邮件地址在运行开始时会收到通知。 如果在创建或重置作业时未指定该列表，该列表将会为空，即，将不会向任何地址发送通知。                                                                                                                                                                                          |
| on_success                         | 一个由 `STRING` 构成的数组       | 一个电子邮件地址列表，其中的电子邮件地址在运行成功完成时会收到通知。 如果在运行结束时 `life_cycle_state` 为 `TERMINATED` 且 result_state 为 `SUCCESSFUL`，则该运行会被视为已成功完成。 如果在创建或重置作业时未指定该列表，该列表将会为空，即，将不会向任何地址发送通知。                                    |
| on_failure                         | 一个由 `STRING` 构成的数组       | 一个电子邮件地址列表，其中的电子邮件地址在运行未成功完成时会收到通知。 如果运行在结束时有 `INTERNAL_ERROR`<br>`life_cycle_state`，或者有 `SKIPPED`、`FAILED` 或 `TIMED_OUT` result_state，则该运行会被视为未成功完成。  如果在创建或重置作业时未指定该列表，该列表将会为空，即，将不会向任何地址发送通知。 |
| no_alert_for_skipped_runs          | `BOOL`                     | 如果为 true，则在该运行被跳过的情况下不会向 `on_failure` 中指定的收件人发送电子邮件。                                                                                                                                                                                                                                                                    |

### <a name="jobsettings"></a><a id="jobsettings"> </a><a id="jobsjobsettings"> </a>JobSettings

> [!IMPORTANT]
>
> * 在新作业群集上运行作业时，该作业会被视为遵从作业计算定价标准的作业计算（自动化）工作负载。
> * 在现有的通用群集上运行作业时，该作业会被视为遵从通用计算定价标准的通用计算（交互式）工作负载。

作业的设置。 可以使用 `resetJob` 方法来更新这些设置。

| 字段名称                                                                | 类型                                                                                                                                                         | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| existing_cluster_id 或 new_cluster                                        | `STRING` 或 [NewCluster](#jobsclusterspecnewcluster)                                                                                                         | 如果是 existing_cluster_id，则该项是将用于此作业的所有运行的现有群集的 ID。 在现有群集上运行作业时，如果该群集停止响应，则可能需要手动重启该群集。 建议在新群集上运行作业，以获得更高的可靠性。<br><br>如果是 new_cluster，则该项是将为每个运行创建的群集的说明。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| notebook_task 或 spark_jar_task 或 spark_python_task 或 spark_submit_task | [NotebookTask](#jobsnotebooktask) 或 [SparkJarTask](#jobssparkjartask) 或 [SparkPythonTask](#jobssparkpythontask) 或 [SparkSubmitTask](#jobssparksubmittask) | 如果是 notebook_task，则表明此作业应该运行笔记本。 此字段不能与 spark_jar_task 一起指定。<br><br>如果是 spark_jar_task，则表明此作业应该运行 JAR。<br><br>如果是 spark_python_task，则表明此作业应该运行 Python 文件。<br><br>如果是 spark_submit_task，则表明此作业应该运行 spark-submit 脚本。                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| name                                                                      | `STRING`                                                                                                                                                     | 可选的作业名称。 默认值是 `Untitled`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| 库                                                                 | 一个由[库](libraries.md#managedlibrarieslibrary)构成的数组                                                                                                  | 可选的库列表，这些库会安装在将要执行该作业的群集上。 默认值为空列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| email_notifications                                                       | [JobEmailNotifications](#jobsjobsettingsjobemailnotifications)                                                                                               | 可选的一组电子邮件地址，在此作业的运行开始或完成时，以及在删除此作业时，这些电子邮件地址将会收到通知。 默认行为是不发送任何电子邮件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| timeout_seconds                                                           | `INT32`                                                                                                                                                      | 可选的超时设置，应用于此作业的每个运行。 默认行为是没有任何超时。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| max_retries                                                               | `INT32`                                                                                                                                                      | 对未成功的运行进行重试的最大次数，可选。 如果运行在完成时其以下状态为 `FAILED`，则会被视为未成功：result_state 或<br>`INTERNAL_ERROR`<br>`life_cycle_state`. 值 -1 表示要无限次重试，而值 0 则表示从不重试。 默认行为是从不重试。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| min_retry_interval_millis                                                 | `INT32`                                                                                                                                                      | 可选的最小时间间隔（介于两次尝试之间），以毫秒为单位。 默认行为是立即重试未成功的运行。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| retry_on_timeout                                                          | `BOOL`                                                                                                                                                       | 一个可选的策略，用于指定在作业超时是否重试该作业。默认行为是在超时发生时不重试。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| schedule                                                                  | [CronSchedule](#jobscronschedule)                                                                                                                            | 此作业的可选定期计划。 默认行为是该作业将只在通过以下方式被触发时运行：在作业 UI 中单击“立即运行”，或将 API 请求发送到<br>`runNow`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| max_concurrent_runs                                                       | `INT32`                                                                                                                                                      | 允许的作业最大并发运行数，可选。<br><br>如果希望能够以并发方式执行同一作业的多个运行，请设置此值。 设置此值适用于这样的情形：例如，如果你按计划频繁触发作业并希望允许连续的运行彼此重叠，或者，如果你希望触发多个在输入参数方面有区别的运行。<br><br>此设置只影响新的运行。 例如，假定作业的并发数为 4，并且存在 4 个并发的活动运行。 那么，将该并发数设置为 3 则不会终止任何活动的运行。 但是，从此之后，除非活动的运行少于 3 个，否则将会跳过新的运行。<br><br>此值不能超过 1000。 将此值设置为 0 会导致跳过所有新的运行。 默认行为是只允许 1 个并发运行。 |

### <a name="jobtask"></a><a id="jobsjobtask"> </a><a id="jobtask"> </a>JobTask

| 字段名称                                                                | 类型                                                                                                                                                         | 描述                                                                                                                                                                                                                                                                                                                                                                  |
|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| notebook_task 或 spark_jar_task 或 spark_python_task 或 spark_submit_task | [NotebookTask](#jobsnotebooktask) 或 [SparkJarTask](#jobssparkjartask) 或 [SparkPythonTask](#jobssparkpythontask) 或 [SparkSubmitTask](#jobssparksubmittask) | 如果是 notebook_task，则表明此作业应该运行笔记本。 此字段不能与 spark_jar_task 一起指定。<br><br>如果是 spark_jar_task，则表明此作业应该运行 JAR。<br><br>如果是 spark_python_task，则表明此作业应该运行 Python 文件。<br><br>如果是 spark_submit_task，则表明此作业应该运行 spark-submit 脚本。 |

### <a name="newcluster"></a><a id="jobsclusterspecnewcluster"> </a><a id="newcluster"> </a>NewCluster

| 字段名称                           | 类型                                                                   | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| num_workers 或 autoscale             | `INT32` 或 [AutoScale](clusters.md#clusterautoscale)                   | 如果是 num_workers，则此项为此群集应该具有的工作器节点数。 一个群集有一个 Spark 驱动程序和 num_workers 个执行程序用于总共 (num_workers + 1) 个 Spark 节点。<br><br>注意：在读取群集的属性时，此字段反映的是所需的工作器数，而不是当前实际的工作器数。 例如，如果将群集的大小从 5 个工作器调整到 10 个工作器，此字段将会立即更新，以反映 10 个工作器的目标大小，而 spark_info 中列出的工作器会随着新节点的预配，逐渐从 5 个增加到 10 个。<br><br>如果是 autoscale，则会需要参数，以便根据负载自动纵向扩展或缩减群集。          |
| spark_version                        | `STRING`                                                               | 群集的 Spark 版本。 可以通过使用[运行时版本](clusters.md#clusterclusterservicelistsparkversions) API 调用来检索可用 Spark 版本的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| spark_conf                           | [SparkConfPair](clusters.md#clustersparkconfpair)                      | 一个对象，其中包含一组可选的由用户指定的 Spark 配置键值对。 你也可以分别通过以下属性，将额外 JVM 选项的字符串传入到驱动程序和执行程序：<br>`spark.driver.extraJavaOptions` 和 `spark.executor.extraJavaOptions`。<br><br>示例 Spark 配置：<br>`{"spark.speculation": true, "spark.streaming.ui.retainedBatches": 5}` 或<br>`{"spark.driver.extraJavaOptions": "-verbose:gc -XX:+PrintGCDetails"}`                                                                                                                                                                                                                                                      |
| node_type_id                         | `STRING`                                                               | 此字段通过单个值对提供给此群集中的每个 Spark 节点的资源进行编码。 例如，可以针对内存密集型或计算密集型的工作负载来预配和优化 Spark 节点。通过使用[列出节点类型](clusters.md#clusterclusterservicelistnodetypes) API 调用可以检索可用节点类型的列表。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                          |
| driver_node_type_id                  | `STRING`                                                               | Spark 驱动程序的节点类型。 此字段为可选；如果未设置，驱动程序节点类型会被设置为与上面定义的 `node_type_id` 相同的值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| custom_tags                          | [ClusterTag](clusters.md#clusterclustertag)                            | 一个对象，其中包含群集资源的一组标记。 Azure Databricks 会使用这些标记以及 default_tags 来标记所有的群集资源。<br><br>**注意**：<br><br>* 旧版节点类型（如计算优化和内存优化）不支持标记<br>* Databricks 最多允许 45 个自定义标记                                                                                                                                                                                                                                                                                                                                                                                             |
| cluster_log_conf                     | [ClusterLogConf](clusters.md#clusterclusterlogconf)                    | 用于将 Spark 日志传递到长期存储目标的配置。 对于一个群集，只能指定一个目标。 如果提供该配置，则会每隔 `5 mins` 向目标发送一次日志。 驱动程序日志的目标是 `<destination>/<cluster-id>/driver`，而执行程序日志的目标是 `<destination>/<cluster-id>/executor`。                                                                                                                                                                                                                                                                                                                                |
| init_scripts                         | 一个由 [InitScriptInfo](clusters.md#clusterclusterinitscriptinfo) 构成的数组 | 用于存储初始化脚本的配置。  可以指定任意数量的脚本。 这些脚本会按照所提供的顺序依次执行。 如果指定了 `cluster_log_conf`，初始化脚本日志将会发送到<br>`<destination>/<cluster-id>/init_scripts`.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| spark_env_vars                       | [SparkEnvPair](clusters.md#clustersparkenvpair)                        | 一个对象，其中包含一组可选的由用户指定的环境变量键值对。 在启动驱动程序和工作器时，(X,Y) 形式的键值对会按原样导出（即<br>`export X='Y'`）。<br><br>若要额外指定一组 `SPARK_DAEMON_JAVA_OPTS`，建议将其追加到 `$SPARK_DAEMON_JAVA_OPTS`，如以下示例中所示。 这样可确保也包含所有默认的 databricks 托管环境变量。<br><br>Spark 环境变量示例：<br>`{"SPARK_WORKER_MEMORY": "28000m", "SPARK_LOCAL_DIRS": "/local_disk0"}` 或<br>`{"SPARK_DAEMON_JAVA_OPTS": "$SPARK_DAEMON_JAVA_OPTS -Dspark.shuffle.service.enabled=true"}` |
| enable_elastic_disk                  | `BOOL`                                                                 | 自动缩放本地存储：启用后，此群集在其 Spark 工作器磁盘空间不足时会动态获取更多磁盘空间。 有关详细信息，请参阅[自动缩放本地存储](../../../clusters/configure.md#autoscaling-local-storage-azure)。                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| instance_pool_id                     | `STRING`                                                               | 群集所属的实例池的可选 ID。 有关详细信息，请参阅[实例池 API](instance-pools.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

### <a name="notebookoutput"></a><a id="jobsnotebooktasknotebookoutput"> </a><a id="notebookoutput"> </a>NotebookOutput

| 字段名称     | 类型           | 描述                                                                                                                                                                                                                                                                                                                                          |
|----------------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| result         | `STRING`       | 传递到 [dbutils.notebook.exit()](../../../notebooks/notebook-workflows.md#notebook-workflows-exit) 的值。 Azure Databricks 将此 API 限制为返回该值的前 1 MB。 若要返回更大的结果，你的作业可以将结果存储在云存储服务中。 如果从未调用 `dbutils.notebook.exit()`，此字段将不会存在。 |
| 已截断      | `BOOLEAN`      | 结果是否已截断。                                                                                                                                                                                                                                                                                                             |

### <a name="notebooktask"></a><a id="jobsnotebooktask"> </a><a id="notebooktask"> </a>NotebookTask

所有输出单元均受大小 8MB 的约束。 如果单元的输出具有更大的大小，该运行的剩余部分将会被取消，并且该运行将会被标记为失败。 在这种情况下，可能还会缺少其他单元中的一些内容输出。 如果需要帮助查找超出限制的单元，请针对通用群集运行该笔记本，并使用这项[笔记本自动保存技术](/databricks/kb/notebooks/notebook-autosave)。

| 字段名称          | 类型                                 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|---------------------|--------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| notebook_path       | `STRING`                             | 要在 Azure Databricks 工作区中运行的笔记本的绝对路径。 此路径必须以斜杠开头。 此字段为必需字段。                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| revision_timestamp  | `LONG`                               | 笔记本的修订的时间戳。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| base_parameters     | [ParamPair](#jobsparampair) 的映射 | 要用于此作业每个运行的基参数。 如果该运行由指定了参数的 `run-now` 调用启动，将会合并这两个参数映射。 如果在 `base_parameters` 和 `run-now` 中指定了相同密钥，将会使用来自 `run-now` 的值。<br><br>如果笔记本采用未在作业的 `base_parameters` 或 `run-now` 重写参数中指定的参数，则将会使用笔记本中的默认值。<br><br>请使用 [dbutils.widgets.get](../../databricks-utils.md#dbutils-widgets) 来检索笔记本中的这些参数。 |

### <a name="parampair"></a><a id="jobsparampair"> </a><a id="parampair"> </a>ParamPair

基于名称的参数，用于运行笔记本任务的作业。

> [!IMPORTANT]
>
> 此数据结构中的字段只接受拉丁字符（ASCII 字符集）。 使用非 ASCII 字符将会返回错误。 例如，中文、日文汉字和表情符号都属于无效的非 ASCII 字符。

| 类型           | 描述                                                                                                     |
|----------------|-----------------------------------------------------------------------------------------------------------------|
| `STRING`       | 参数名称。 传递到 [dbutils.widgets.get](../../databricks-utils.md#dbutils-widgets) 以检索值。 |
| `STRING`       | 参数值。                                                                                                |

### <a name="run"></a><a id="jobsrun"> </a><a id="run"> </a>运行

有关运行的所有信息（其输出除外）。 可以使用 `getRunOutput` 方法单独检索输出。

| 字段名称                  | 类型                                    | 描述                                                                                                                                                                                                                                                                                                                        |
|-----------------------------|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| job_id                      | `INT64`                                 | 包含此运行的作业的规范标识符。                                                                                                                                                                                                                                                                        |
| run_id                      | `INT64`                                 | 该运行的规范标识符。 此 ID 在所有作业的所有运行中都是唯一的。                                                                                                                                                                                                                                                |
| creator_user_name           | `STRING`                                | 创建者用户名。 如果已将该用户删除，响应中将不会包含此字段。                                                                                                                                                                                                                          |
| number_in_job               | `INT64`                                 | 此运行在该作业的所有运行中的序列号。 此值从 1 开始。                                                                                                                                                                                                                                                 |
| original_attempt_run_id     | `INT64`                                 | 如果此运行是对之前某次运行尝试的重试，则此字段会包含原始尝试的 run_id；否则此字段与本次运行的 run_id 相同。                                                                                                                                                                                    |
| state                       | [RunState](#jobsrunstate)               | 运行的结果和生命周期状态。                                                                                                                                                                                                                                                                                        |
| schedule                    | [CronSchedule](#jobscronschedule)       | 触发此运行的 cron 计划（如果此运行已由定期计划程序触发）。                                                                                                                                                                                                                                           |
| task                        | [JobTask](#jobsjobtask)                 | 由该运行执行的任务（如果有）。                                                                                                                                                                                                                                                                                             |
| cluster_spec                | [ClusterSpec](#jobsclusterspec)         | 创建此运行时作业的群集规范的快照。                                                                                                                                                                                                                                                           |
| cluster_instance            | [ClusterInstance](#jobsclusterinstance) | 用于此运行的群集。 如果将该运行指定为使用新群集，则在作业服务已为该运行请求群集后，将会设置此字段。                                                                                                                                                                     |
| overriding_parameters       | [RunParameters](#jobsrunparameters)     | 用于此运行的参数。                                                                                                                                                                                                                                                                                                  |
| start_time                  | `INT64`                                 | 启动此运行的时间，以epoch 毫秒表示（自 UTC 1970 年 1 月 1 日起的毫秒数）。 此时间可能不是作业任务开始执行的时间，例如，如果该作业已计划在新群集上运行，则此时间是发出群集创建调用的时间。                                                   |
| setup_duration              | `INT64`                                 | 设置该群集所花费的时间（以毫秒为单位）。 对于在新群集上运行的运行，此时间是群集创建时间，对于在现有群集上运行的运行，此时间应该很短。                                                                                                                                |
| execution_duration          | `INT64`                                 | 执行 JAR 或笔记本中的命令（直到这些命令已完成、失败、超时、被取消或遇到意外错误）所花费的时间（以毫秒为单位）。                                                                                                                                                       |
| cleanup_duration            | `INT64`                                 | 终止该群集并清理任何中间结果等等所花费的时间（以毫秒为单位）。该运行的总持续时间为 setup_duration、execution_duration 以及 cleanup_duration 之和。                                                                                                            |
| 触发器                     | [TriggerType](#jobstriggertype)         | 触发此运行的触发器的类型，例如，定期计划或一次性运行。                                                                                                                                                                                                                                              |
| run_name                    | `STRING`                                | 可选的作业名称。 默认值是 `Untitled`。   允许的最大长度为采用 UTF-8 编码的 4096 个字节。                                                                                                                                                                                                       |
| run_page_url                | `STRING`                                | 运行的详细信息页的 URL。                                                                                                                                                                                                                                                                                             |
| run_type                    | `STRING`                                | 运行的类型。<br><br>* `JOB_RUN` - 正常的作业运行。 使用[立即运行](#jobsjobsservicerunnow)创建的运行。<br>* `WORKFLOW_RUN` - 工作流运行。   使用 [dbutils.notebook.run](../../databricks-utils.md#dbutils-workflow) 创建的运行。<br>* `SUBMIT_RUN` - 提交运行。 使用[立即运行](#jobsjobsservicerunnow)创建的运行。 |

### <a name="runparameters"></a><a id="jobsrunparameters"> </a><a id="runparameters"> </a>RunParameters

适用于此运行的参数。 在 `run-now` 请求中，只应根据作业任务类型的不同指定 jar_params、`python_params` 或 notebook_params 之一。
具有 Spark JAR 任务或 Python 任务的作业采用基于位置的参数的列表，而具有笔记本任务的作业则采用键值映射。

| 字段名称              | 类型                                 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-------------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| jar_params              | 一个由 `STRING` 构成的数组                 | 具有 Spark JAR 任务的作业的参数的列表，例如 `"jar_params": ["john doe", "35"]`。 这些参数将用于调用 Spark JAR 任务中指定的 main 类的 main 函数。 如果未在调用 `run-now` 时指定，该项将默认为空列表。 jar_params 不能与 notebook_params 一起指定。 此字段的 JSON 表示形式（即 `{"jar_params":["john doe","35"]}`）不能超过 10,000 字节。                                                                                                                                                                                                            |
| notebook_params         | [ParamPair](#jobsparampair) 的映射 | 具有笔记本任务的作业的从键到值的映射，例如<br>`"notebook_params": {"name": "john doe", "age":  "35"}`. 该映射会传递到笔记本，并且可通过 [dbutils.widgets.get](../../databricks-utils.md#dbutils-widgets) 函数访问。<br><br>如果未在调用 `run-now` 时指定，已触发的运行会使用该作业的基参数。<br><br>notebook_params 不能与 jar_params 一起指定。<br><br>此字段的 JSON 表示形式（即<br>`{"notebook_params":{"name":"john doe","age":"35"}}`）不能超过 10,000 字节。                                                                                       |
| python_params           | 一个由 `STRING` 构成的数组                 | 具有 Python 任务的作业的参数列表，例如 `"python_params": ["john doe", "35"]`。 这些参数会作为命令行参数传递到 Python 文件。 如果在调用 `run-now` 时指定，则此项会覆盖作业设置中指定的参数。 此字段的 JSON 表示形式（即 `{"python_params":["john doe","35"]}`）不能超过 10,000 字节。<br><br>> [!IMPORTANT] > > 这些参数只接受拉丁字符（ASCII 字符集）。 > 使用非 ASCII 字符将会返回错误。 例如，中文、日文汉字和表情符号 > 都属于无效的非 ASCII 字符。                                                     |
| spark_submit_params     | 一个由 `STRING` 构成的数组                 | 具有“Spark 提交”任务的作业的参数列表，例如<br>`"spark_submit_params": ["--class", "org.apache.spark.examples.SparkPi"]`. 这些参数会作为命令行参数传递到 spark-submit 脚本。 如果在调用 `run-now` 时指定，则此项会覆盖作业设置中指定的参数。 此字段的 JSON 表示形式（即 `{"python_params":["john doe","35"]}`）不能超过 10,000 字节。<br><br>> [!IMPORTANT] > > 这些参数只接受拉丁字符（ASCII 字符集）。 > 使用非 ASCII 字符将会返回错误。 例如，中文、日文汉字和表情符号 > 都属于无效的非 ASCII 字符。 |

### <a name="runstate"></a><a id="jobsrunstate"> </a><a id="runstate"> </a>RunState

| 字段名称           | 类型                                        | 描述                                                                                                                                                                         |
|----------------------|---------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| life_cycle_state     | [RunLifeCycleState](#jobsrunlifecyclestate) | 运行在运行生命周期中当前所处位置的说明。 此字段在响应中始终可用。                                                                     |
| result_state         | [RunResultState](#jobsrunresultstate)       | 运行的结果状态。 如果此字段不可用，响应将不会包含此字段。 有关 result_state 可用性的详细信息，请参阅 [RunResultState](#runresultstate)。 |
| state_message        | `STRING`                                    | 当前状态的描述性消息。 此字段是非结构化的，且其确切格式随时可能发生变更。                                                                 |

### <a name="sparkjartask"></a><a id="jobssparkjartask"> </a><a id="sparkjartask"> </a>SparkJarTask

| 字段名称          | 类型                       | 描述                                                                                                                                                                                                                                               |
|---------------------|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| jar_uri             | `STRING`                   | 自 2016 年 4 月起已弃用。 改为通过 `libraries` 字段提供 `jar`。 有关示例，请参阅[创建](#create)。                                                                                                                                   |
| main_class_name     | `STRING`                   | 类的全名，包含要执行的主要方法。 此类必须包含在作为库提供的 JAR 中。<br><br>该代码应使用 `SparkContext.getOrCreate` 来获取 Spark 上下文；否则，作业的运行将会失败。 |
| parameters          | 一个由 `STRING` 构成的数组       | 传递给 main 方法的参数。                                                                                                                                                                                                                     |

### <a name="sparkpythontask"></a><a id="jobssparkpythontask"> </a><a id="sparkpythontask"> </a>SparkPythonTask

| 字段名称      | 类型                       | 描述                                                                                   |
|-----------------|----------------------------|-----------------------------------------------------------------------------------------------|
| python_file     | `STRING`                   | 要执行的 Python 文件的 URI。 支持 DBFS 路径。 此字段为必需字段。  |
| parameters      | 一个由 `STRING` 构成的数组       | 传递到 Python 文件的命令行参数。                                            |

### <a name="sparksubmittask"></a><a id="jobssparksubmittask"> </a><a id="sparksubmittask"> </a>SparkSubmitTask

> [!IMPORTANT]
>
> * 只能对新群集调用“Spark 提交”任务。
> * 在 new_cluster 规范中，不支持 `libraries` 和 `spark_conf`。 请改为使用 `--jars` 和 `--py-files` 来添加 Java 和 Python 库，使用 `--conf` 来设置 Spark 配置。
> * `master`、`deploy-mode` 和 `executor-cores` 由 Azure Databricks 自动配置；你无法在参数中指定它们。
> * 默认情况下，“Spark 提交”作业使用所有可用内存（为 Azure Databricks 服务保留的内存除外）。 可将 `--driver-memory` 和 `--executor-memory` 设置为较小的值，以留出一些空间作为堆外内存。
> * `--jars`、`--py-files`、`--files` 参数支持 DBFS 路径。

例如，假定 JAR 上传到了 DBFS，则可通过设置以下参数来运行 `SparkPi`。

```json
{
  "parameters": [
    "--class",
    "org.apache.spark.examples.SparkPi",
    "dbfs:/path/to/examples.jar",
    "10"
  ]
}
```

| 字段名称     | 类型                       | 描述                                                      |
|----------------|----------------------------|------------------------------------------------------------------|
| parameters     | 一个由 `STRING` 构成的数组       | 传递到 spark-submit 的命令行参数。                  |

### <a name="viewitem"></a><a id="jobsviewitem"> </a><a id="viewitem"> </a>ViewItem

导出的内容采用 HTML 格式。 例如，如果要导出的视图是仪表板，则会为每个仪表板返回一个 HTML 字符串。

| 字段名称     | 类型                      | 描述                                                                                                                                        |
|----------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| 内容        | `STRING`                  | 视图的内容。                                                                                                                               |
| name           | `STRING`                  | 视图项的名称。 对于代码视图，该项会是笔记本的名称。 对于仪表板视图，该项会是仪表板的名称。 |
| type           | [ViewType](#jobsviewtype) | 视图项的类型。                                                                                                                             |

### <a name="runlifecyclestate"></a><a id="jobsrunlifecyclestate"> </a><a id="runlifecyclestate"> </a>RunLifeCycleState

运行的生命周期状态。 允许的状态转换为：

* `PENDING` -> `RUNNING` -> `TERMINATING` -> `TERMINATED`
* `PENDING` -> `SKIPPED`
* `PENDING` -> `INTERNAL_ERROR`
* `RUNNING` -> `INTERNAL_ERROR`
* `TERMINATING` -> `INTERNAL_ERROR`

| 状态                  | 描述                                                                                                                                                                                                                                                                               |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `PENDING`              | 已触发该运行。 如果还没有相同作业的活动运行，则会准备群集和执行上下文。 如果已经有相同作业的活动运行，该运行将会立即转换为 `SKIPPED` 状态，而不会再准备任何资源。 |
| `RUNNING`              | 正在执行此运行的任务。                                                                                                                                                                                                                                                   |
| `TERMINATING`          | 此运行的任务已经完成，正在清理群集和执行上下文。                                                                                                                                                                                           |
| `TERMINATED`           | 此运行的任务已经完成，已经清理了群集和执行上下文。 此状态为最终状态。                                                                                                                                                                   |
| `SKIPPED`              | 已中止此运行，因为同一作业以前的运行已处于活动状态。 此状态为最终状态。                                                                                                                                                                                   |
| `INTERNAL_ERROR`       | 一个异常状态，指示作业服务中存在故障（如长时间的网络故障）。 如果新群集上的运行以 `INTERNAL_ERROR` 状态结束，作业服务会尽快终止该群集。 此状态为最终状态。                         |

### <a name="runresultstate"></a><a id="jobsrunresultstate"> </a><a id="runresultstate"> </a>RunResultState

运行的结果状态。

* 如果 `life_cycle_state` = `TERMINATED`：在该运行有任务的情况下，一定会提供结果，并且该状态指示任务的结果。
* 如果 `life_cycle_state` = `PENDING`、`RUNNING` 或 `SKIPPED`，则不会提供结果状态。
* 如果 `life_cycle_state` = `TERMINATING` 或 lifecyclestate = `INTERNAL_ERROR`：在该运行有任务并已成功启动该任务的情况下，将会提供结果状态。

结果状态在提供之后不会再改变。

| 状态        | 描述                                         |
|--------------|-----------------------------------------------------|
| 成功      | 已成功完成任务。                    |
| FAILED       | 任务已完成，但有错误。                   |
| TIMEDOUT     | 该运行在达到超时后已停止。     |
| 已取消     | 该运行已应用户请求而取消。               |

### <a name="triggertype"></a><a id="jobstriggertype"> </a><a id="triggertype"> </a>TriggerType

这些是可以触发运行的触发器的类型。

| 类型         | 描述                                                                                                                                  |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| PERIODIC     | 定期触发运行的计划，如 cron 计划程序。                                                                          |
| ONE_TIME     | 触发单个运行的一次性触发器。 这在你通过 UI 或 API 按需触发单个运行时发生。                        |
| 重试        | 指示一个作为先前失败的运行的重试而被触发的运行。 如果在出现故障时请求重新运行作业，则会发生这种触发。 |

### <a name="viewtype"></a><a id="jobsviewtype"> </a><a id="viewtype"> </a>ViewType

| 类型          | 描述             |
|---------------|-------------------------|
| NOTEBOOK      | 笔记本视图项。     |
| 仪表板     | 仪表板视图项。    |

### <a name="viewstoexport"></a><a id="jobsviewstoexport"> </a><a id="viewstoexport"> </a>ViewsToExport

要导出的视图：代码、所有仪表板，或全部。

| 类型           | 描述                             |
|----------------|-----------------------------------------|
| CODE           | 笔记本的代码视图。              |
| 仪表板     | 笔记本的所有仪表板视图。    |
| ALL            | 笔记本的所有视图。              |