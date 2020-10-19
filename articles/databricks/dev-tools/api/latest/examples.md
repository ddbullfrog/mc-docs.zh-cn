---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/01/2020
title: API 示例 - Azure Databricks
description: 通过示例了解如何使用 Databricks REST API 2.0。
ms.openlocfilehash: b3cd957108c8b1a537d53078475e08e360c5099e
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937683"
---
# <a name="api-examples"></a>API 示例

本文包含的示例演示了如何使用 Azure Databricks REST API 2.0。

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

## <a name="authentication"></a>身份验证

若要了解如何对 REST API 进行身份验证，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](authentication.md)和[使用 Azure Active Directory 令牌进行身份验证](aad/index.md)。

本文中的示例假设你使用的是 Azure Databricks [个人访问令牌](authentication.md)。 在以下示例中，将 `<your-token>` 替换为你的个人访问令牌。 `curl` 示例假设你将 Azure Databricks API 凭据存储在 [.netrc](authentication.md#netrc) 中。 Python 示例使用 [Bearer](authentication.md#bearer) 身份验证。 尽管示例显示了将令牌存储在代码中，但为在 Azure Databricks 中安全地利用凭据，建议按[机密管理](../../../security/secrets/index.md#secrets-user-guide)用户指南进行操作。

有关按[使用 Azure Active Directory 令牌进行身份验证](aad/index.md)进行操作的示例，请参阅该部分中的文章。

## <a name="get-a-gzipped-list-of-clusters"></a><a id="get-a-gzipped-list-of-clusters"> </a><a id="gzip"> </a>获取群集的 gzip 压缩列表

```bash
curl -n -H "Accept-Encoding: gzip" https://<databricks-instance>/api/2.0/clusters/list > clusters.gz
```

## <a name="upload-a-big-file-into-dbfs"></a><a id="dbfs-large-files"> </a><a id="upload-a-big-file-into-dbfs"> </a>将大文件上传到 DBFS

单个 API 调用上传的数据量不能超过 1MB。 若要将超过 1MB 的文件上传到 DBFS，请使用由 `create`、`addBlock` 和 `close` 组合而成的流式处理 API。

以下是有关如何使用 Python 执行此操作的示例。

```python
import json
import requests
import base64

DOMAIN = '<databricks-instance>'
TOKEN = '<your-token>'
BASE_URL = 'https://%s/api/2.0/dbfs/' % (DOMAIN)

def dbfs_rpc(action, body):
  """ A helper function to make the DBFS API request, request/response is encoded/decoded as JSON """
  response = requests.post(
    BASE_URL + action,
    headers={'Authorization': 'Bearer %s' % TOKEN },
    json=body
  )
  return response.json()

# Create a handle that will be used to add blocks
handle = dbfs_rpc("create", {"path": "/temp/upload_large_file", "overwrite": "true"})['handle']
with open('/a/local/file') as f:
  while True:
    # A block can be at most 1MB
    block = f.read(1 << 20)
    if not block:
        break
    data = base64.standard_b64encode(block)
    dbfs_rpc("add-block", {"handle": handle, "data": data})
# close the handle to finish uploading
dbfs_rpc("close", {"handle": handle})
```

## <a name="create-a-python-3-cluster-databricks-runtime-55-lts"></a><a id="create-a-python-3-cluster-databricks-runtime-55-lts"> </a><a id="python-3-example"> </a>创建 Python 3 群集 (Databricks Runtime 5.5 LTS)

> [!NOTE]
>
> 在 Databricks Runtime 6.0 及更高版本和带有 Conda 的 Databricks Runtime 中，Python 3 是默认的 Python 版本。

以下示例显示了如何使用 Databricks REST API 和 Python HTTP 库 [Requests](https://requests.readthedocs.io/en/master/)来启动 [Python 3](../../../clusters/configure.md#python-3) 群集：

```python
import requests

DOMAIN = '<databricks-instance>'
TOKEN = '<your-token>'

response = requests.post(
  'https://%s/api/2.0/clusters/create' % (DOMAIN),
  headers={'Authorization': 'Bearer %s' % TOKEN},
  json={
    "cluster_name": "my-cluster",
    "spark_version": "5.5.x-scala2.11",
    "node_type_id": "Standard_D3_v2",
    "spark_env_vars": {
      "PYSPARK_PYTHON": "/databricks/python3/bin/python3",
    }
  }
)

if response.status_code == 200:
  print(response.json()['cluster_id'])
else:
  print("Error launching cluster: %s: %s" % (response.json()["error_code"], response.json()["message"]))
```

## <a name="create-a-high-concurrency-cluster"></a><a id="create-a-high-concurrency-cluster"> </a><a id="high-concurrency-example"> </a>创建高并发群集

以下示例显示了如何使用 Databricks REST API 启动[高并发](../../../clusters/configure.md#high-concurrency)模式群集：

```bash
curl -n -X POST -H 'Content-Type: application/json' -d '{
  "cluster_name": "high-concurrency-cluster",
  "spark_version": "6.3.x-scala2.11",
  "node_type_id": "Standard_D3_v2",
  "spark_conf":{
        "spark.databricks.cluster.profile":"serverless",
        "spark.databricks.repl.allowedLanguages":"sql,python,r"
     },
   "custom_tags":{
        "ResourceClass":"Serverless"
     },
       "autoscale":{
        "min_workers":1,
        "max_workers":2
     },
  "autotermination_minutes":10
}' https://<databricks-instance>/api/2.0/clusters/create
```

## <a name="jobs-api-examples"></a><a id="jobs-api-example"> </a><a id="jobs-api-examples"> </a>作业 API 示例

本部分介绍如何创建 Python、spark submit 和 JAR 作业，以及如何运行 JAR 作业并查看其输出。

### <a name="create-a-python-job"></a>创建 Python 作业

此示例演示如何创建 Python 作业。 它使用 Apache Spark Python Spark Pi 估算。

1. 下载包含该示例的 [Python 文件](../../../_static/examples/pi.py)，然后使用 [Databricks CLI](../../cli/index.md) 将其上传到 [Databricks 文件系统 (DBFS)](../../../data/databricks-file-system.md)。

   ```bash
   dbfs cp pi.py dbfs:/docs/pi.py
   ```

2. 创建作业。 以下示例演示如何使用 [Databricks Runtime](../../../runtime/dbr.md) 和 [Databricks Light](../../../runtime/light.md) 创建作业。

   **Databricks Runtime**

   ```bash
   curl -n -X POST -H 'Content-Type: application/json' -d \
   '{
     "name": "SparkPi Python job",
     "new_cluster": {
       "spark_version": "6.2.x-scala2.11",
       "node_type_id": "Standard_D3_v2",
       "num_workers": 2
     },
     "spark_python_task": {
       "python_file": "dbfs:/pi.py",
       "parameters": [
         "10"
       ]
     }
   }' https://<databricks-instance>/api/2.0/jobs/create
   ```

   **Databricks Light**

   ```bash
   curl -n -X POST -H 'Content-Type: application/json' -d
   '{
       "name": "SparkPi Python job",
       "new_cluster": {
        "spark_version": "apache-spark-2.4.x-scala2.11",
        "node_type_id": "Standard_D3_v2",
        "num_workers": 2
       },
       "spark_python_task": {
        "python_file": "dbfs:/pi.py",
        "parameters": [
          "10"
        ]
     }
   }' https://<databricks-instance>/api/2.0/jobs/create
   ```

### <a name="create-a-spark-submit-job"></a><a id="create-a-spark-submit-job"> </a><a id="spark-submit-api-example"> </a>创建 spark-submit 作业

此示例演示如何创建 spark-submit 作业。  它使用 Apache Spark [SparkPi 示例](https://raw.githubusercontent.com/apache/spark/master/examples/src/main/scala/org/apache/spark/examples/SparkPi.scala)。

1. 下载包含该示例的 [JAR](../../../_static/examples/SparkPi-assembly-0.1.jar)，然后使用 [Databricks CLI](../../cli/index.md) 将 JAR 上传到 [Databricks 文件系统 (DBFS)](../../../data/databricks-file-system.md)。

   ```bash
   dbfs cp SparkPi-assembly-0.1.jar dbfs:/docs/sparkpi.jar
   ```

2. 创建作业。

   ```bash
   curl -n \
   -X POST -H 'Content-Type: application/json' \
   -d '{
        "name": "SparkPi spark-submit job",
        "new_cluster": {
          "spark_version": "5.2.x-scala2.11",
          "node_type_id": "Standard_DS3_v2",
          "num_workers": 2
          },
       "spark_submit_task": {
          "parameters": [
            "--class",
            "org.apache.spark.examples.SparkPi",
            "dbfs:/docs/sparkpi.jar",
            "10"
         ]
       }
   }' https://<databricks-instance>/api/2.0/jobs/create
   ```

### <a name="create-and-run-a-spark-submit-job-for-r-scripts"></a><a id="create-and-run-a-spark-submit-job-for-r-scripts"> </a><a id="spark-submit-api-example-r"> </a>创建并运行适用于 R 脚本的 spark-submit 作业

此示例演示如何创建 spark-submit 作业以运行 R 脚本。

1. 使用 [Databricks CLI](../../cli/index.md) 将 R 文件上传到 [Databricks 文件系统 (DBFS)](../../../data/databricks-file-system.md)。

   ```bash
   dbfs cp your_code.R dbfs:/path/to/your_code.R
   ```

   如果代码使用 SparkR，则必须先安装包。 Databricks Runtime 包含 SparkR 源代码。 如以下示例所示，从其本地目录安装 SparkR 包：

   ```r
   install.packages("/databricks/spark/R/pkg", repos = NULL)
   library(SparkR)

   sparkR.session()
   n <- nrow(createDataFrame(iris))
   write.csv(n, "/dbfs/path/to/num_rows.csv")
   ```

   <!--important: Do not install SparkR from CRAN.-->

   Databricks Runtime 从 CRAN 安装最新版的 sparklyr。  如果代码使用 sparklyr，则你必须在 `spark_connect` 中指定 Spark 主 URL。 若要构成 Spark 主 URL，请使用 `SPARK_LOCAL_IP` 环境变量来获取 IP，然后使用默认端口 7077。 例如：

   ```r
   library(sparklyr)

   master <- paste("spark://", Sys.getenv("SPARK_LOCAL_IP"), ":7077", sep="")
   sc <- spark_connect(master)
   iris_tbl <- copy_to(sc, iris)
   write.csv(iris_tbl, "/dbfs/path/to/sparklyr_iris.csv")
   ```

2. 创建作业。

   ```bash
   curl -n \
   -X POST -H 'Content-Type: application/json' \
   -d '{
        "name": "R script spark-submit job",
        "new_cluster": {
          "spark_version": "5.5.x-scala2.11",
          "node_type_id": "Standard_DS3_v2",
          "num_workers": 2
          },
       "spark_submit_task": {
          "parameters": [ "dbfs:/path/to/your_code.R" ]
          }
   }' https://<databricks-instance>/api/2.0/jobs/create
   ```

   这将返回 `job-id`，随后可使用它来运行作业。

3. 使用 `job-id` 运行作业。

   ```bash
   curl -n \
   -X POST -H 'Content-Type: application/json' \
   -d '{ "job_id": <job-id> }' https://<databricks-instance>/api/2.0/jobs/run-now
   ```

### <a name="create-and-run-a-jar-job"></a><a id="create-and-run-a-jar-job"> </a><a id="spark-jar-job"> </a>创建并运行 JAR 作业

此示例说明如何创建并运行 JAR 作业。 它使用 Apache Spark [SparkPi 示例](https://raw.githubusercontent.com/apache/spark/master/examples/src/main/scala/org/apache/spark/examples/SparkPi.scala)。

1. 下载包含该示例的 [JAR](../../../_static/examples/SparkPi-assembly-0.1.jar)。
2. 使用 API 将 JAR 上传到 Azure Databricks 实例：

   ```bash
   curl -n \
   -F filedata=@"SparkPi-assembly-0.1.jar" \
   -F path="/docs/sparkpi.jar" \
   -F overwrite=true \
   https://<databricks-instance>/api/2.0/dbfs/put
   ```

   调用成功会返回 `{}`。 否则，你会看到错误消息。

3. 创建作业前，获取所有 Spark 版本的列表。

   ```bash
   curl -n https://<databricks-instance>/api/2.0/clusters/spark-versions
   ```

   本示例使用 `5.2.x-scala2.11`。 有关 Spark 群集版本的更多信息，请参阅 [Runtime 版本字符串](index.md#programmatic-version)。

4. 创建作业。 JAR 指定为库，主类名在 Spark JAR 任务中进行引用。

   ```bash
   curl -n -X POST -H 'Content-Type: application/json' \
   -d '{
         "name": "SparkPi JAR job",
         "new_cluster": {
           "spark_version": "5.2.x-scala2.11",
           "node_type_id": "Standard_DS3_v2",
           "num_workers": 2
           },
        "libraries": [{"jar": "dbfs:/docs/sparkpi.jar"}],
        "spark_jar_task": {
           "main_class_name":"org.apache.spark.examples.SparkPi",
           "parameters": "10"
           }
   }' https://<databricks-instance>/api/2.0/jobs/create
   ```

   这将返回 `job-id`，随后可使用它来运行作业。

5. 使用 `run now` 运行作业：

   ```bash
   curl -n \
   -X POST -H 'Content-Type: application/json' \
   -d '{ "job_id": <job-id> }' https://<databricks-instance>/api/2.0/jobs/run-now
   ```

6. 导航到 `https://<databricks-instance>/#job/<job-id>`，你将可以看到作业正在运行。
7. 你还可以使用上个请求返回的信息从 API 对其进行检查。

   ```bash
   curl -n https://<databricks-instance>/api/2.0/jobs/runs/get?run_id=<run-id> | jq
   ```

   这应返回如下所示的输出：

   ```json
   {
     "job_id": 35,
     "run_id": 30,
     "number_in_job": 1,
     "original_attempt_run_id": 30,
     "state": {
       "life_cycle_state": "TERMINATED",
       "result_state": "SUCCESS",
       "state_message": ""
     },
     "task": {
       "spark_jar_task": {
         "jar_uri": "",
         "main_class_name": "org.apache.spark.examples.SparkPi",
         "parameters": [
           "10"
         ],
         "run_as_repl": true
       }
     },
     "cluster_spec": {
       "new_cluster": {
         "spark_version": "5.2.x-scala2.11",
         "node_type_id": "<node-type>",
         "enable_elastic_disk": false,
         "num_workers": 1
       },
       "libraries": [
         {
           "jar": "dbfs:/docs/sparkpi.jar"
         }
       ]
     },
     "cluster_instance": {
       "cluster_id": "0412-165350-type465",
       "spark_context_id": "5998195893958609953"
     },
     "start_time": 1523552029282,
     "setup_duration": 211000,
     "execution_duration": 33000,
     "cleanup_duration": 2000,
     "trigger": "ONE_TIME",
     "creator_user_name": "...",
     "run_name": "SparkPi JAR job",
     "run_page_url": "<databricks-instance>/?o=3901135158661429#job/35/run/1",
     "run_type": "JOB_RUN"
   }
   ```

8. 若要查看作业输出，请访问[作业运行详细信息页](../../../jobs.md#job-run-details)。

   ```
   Executing command, time = 1523552263909.
   Pi is roughly 3.13973913973914
   ```

## <a name="create-cluster-enabled-for-table-access-control-example"></a><a id="cluster-table-acl-example"> </a><a id="create-cluster-enabled-for-table-access-control-example"> </a>有关创建启用了表访问控制的群集的示例

若要创建启用了表访问控制的群集，请在请求正文中指定以下 `spark_conf` 属性：

```bash
curl -X POST https://<databricks-instance>/api/2.0/clusters/create -d'
{
  "cluster_name": "my-cluster",
  "spark_version": "5.2.x-scala2.11",
  "node_type_id": "Standard_DS3_v2",
  "spark_conf": {
    "spark.databricks.acl.dfAclsEnabled":true,
    "spark.databricks.repl.allowedLanguages": "python,sql"
  },
  "num_workers": 1,
  "custom_tags":{
     "costcenter":"Tags",
     "applicationname":"Tags1"
  }
}'
```

## <a name="cluster-log-delivery-examples"></a><a id="cluster-log-delivery-examples"> </a><a id="cluster-log-example"> </a>群集日志传送示例

你可以在 Spark UI 中查看 Spark 驱动程序和执行程序日志，Azure Databricks 也可以将日志传递到 DBFS 目标。
请参阅以下示例。

### <a name="create-a-cluster-with-logs-delivered-to-a-dbfs-location"></a>创建一个群集，其中的日志传递到 DBFS 位置

以下 cURL 命令创建名为 `cluster_log_dbfs` 的群集，并请求 Azure Databricks 将其日志发送到群集 ID 为路径前缀的 `dbfs:/logs`。

```bash
curl -n -X POST -H 'Content-Type: application/json' -d \
'{
  "cluster_name": "cluster_log_dbfs",
  "spark_version": "5.2.x-scala2.11",
  "node_type_id": "Standard_D3_v2",
  "num_workers": 1,
  "cluster_log_conf": {
    "dbfs": {
      "destination": "dbfs:/logs"
    }
  }
}' https://<databricks-instance>/api/2.0/clusters/create
```

响应应包含群集 ID：

```json
{"cluster_id":"1111-223344-abc55"}
```

创建群集后，Azure Databricks 每 5 分钟将日志文件同步到目标。
它将驱动程序日志上传到 `dbfs:/logs/1111-223344-abc55/driver`，将执行程序日志上传到 `dbfs:/logs/1111-223344-abc55/executor`。

### <a name="check-log-delivery-status"></a>检查日志传送状态

可以通过 API 检索包含日志传送状态的群集信息：

```bash
curl -n -H 'Content-Type: application/json' -d \
'{
  "cluster_id": "1111-223344-abc55"
}' https://<databricks-instance>/api/2.0/clusters/get
```

如果成功上传最近一批日志，则响应应仅包含最后一次尝试的时间戳：

```json
{
  "cluster_log_status": {
    "last_attempted": 1479338561
  }
}
```

如果出现错误，错误消息会显示在响应中：

```json
{
  "cluster_log_status": {
    "last_attempted": 1479338561,
    "last_exception": "Exception: Access Denied ..."
  }
}
```

## <a name="workspace-examples"></a><a id="workspace-api-example"> </a><a id="workspace-examples"> </a>工作区示例

下面是使用工作区 API 列出、创建、删除、导出和导入工作区对象以及获取其相关信息的一些示例。

### <a name="list-a-notebook-or-a-folder"></a>列出笔记本或文件夹

以下 cURL 命令列出了工作区中的一个路径。

```bash
curl -n -X GET -H 'Content-Type: application/json' -d \
'{
  "path": "/Users/user@example.com/"
}' https://<databricks-instance>/api/2.0/workspace/list
```

响应应包含状态列表：

```json
{
  "objects": [
    {
     "object_type": "DIRECTORY",
     "path": "/Users/user@example.com/folder"
    },
    {
     "object_type": "NOTEBOOK",
     "language": "PYTHON",
     "path": "/Users/user@example.com/notebook1"
    },
    {
     "object_type": "NOTEBOOK",
     "language": "SCALA",
     "path": "/Users/user@example.com/notebook2"
    }
  ]
}
```

如果该路径是一个笔记本，则响应将包含具有输入笔记本状态的数组。

### <a name="get-information-about-a-notebook-or-a-folder"></a>获取有关笔记本或文件夹的信息

以下 cURL 命令获取工作区中一个路径的状态。

```bash
curl -n  -X GET -H 'Content-Type: application/json' -d \
'{
  "path": "/Users/user@example.com/"
}' https://<databricks-instance>/api/2.0/workspace/get-status
```

响应应包含输入路径的状态：

```json
{
  "object_type": "DIRECTORY",
  "path": "/Users/user@example.com"
}
```

### <a name="create-a-folder"></a>创建一个文件夹

以下 cURL 命令创建文件夹。 它以递归方式创建文件夹，例如 `mkdir -p`。
如果文件夹已存在，它不需执行任何操作即可成功执行。

```bash
curl -n -X POST -H 'Content-Type: application/json' -d \
'{
  "path": "/Users/user@example.com/new/folder"
}' https://<databricks-instance>/api/2.0/workspace/mkdirs
```

如果请求成功，将返回空 JSON 字符串。

### <a name="delete-a-notebook-or-folder"></a>删除笔记本或文件夹

以下 cURL 命令删除笔记本或文件夹。 可以启用 `recursive` 以递归方式删除非空文件夹。

```bash
curl -n -X POST -H 'Content-Type: application/json' -d \
'{
  "path": "/Users/user@example.com/new/folder",
  "recursive": "false"
}' https://<databricks-instance>/api/2.0/workspace/delete
```

如果请求成功，则返回空 JSON 字符串。

### <a name="export-a-notebook-or-folder"></a><a id="export-a-notebook-or-folder"> </a><a id="workspace-api-export-example"> </a>导出笔记本或文件夹

以下 cURL 命令导出笔记本。 可以以下格式导出笔记本：`SOURCE`、`HTML`、`JUPYTER`、`DBC`。 文件夹只能导出为 `DBC`。

```bash
curl -n  -X GET -H 'Content-Type: application/json' \
'{
  "path": "/Users/user@example.com/notebook",
  "format": "SOURCE"
}' https://<databricks-instance>/api/2.0/workspace/export
```

响应包含 base64 编码的笔记本内容。

```json
{
  "content": "Ly8gRGF0YWJyaWNrcyBub3RlYm9vayBzb3VyY2UKcHJpbnQoImhlbGxvLCB3b3JsZCIpCgovLyBDT01NQU5EIC0tLS0tLS0tLS0KCg=="
}
```

或者，可以直接下载导出的笔记本。

```bash
curl -n -X GET "https://<databricks-instance>/api/2.0/workspace/export?format=SOURCE&direct_download=true&path=/Users/user@example.com/notebook"
```

响应将是导出的笔记本内容。

### <a name="import-a-notebook-or-directory"></a><a id="import-a-notebook-or-directory"> </a><a id="workspace-api-import-example"> </a>导入笔记本或目录

以下 cURL 命令在工作区中导入笔记本。 支持多种格式（`SOURCE`、`HTML`、`JUPYTER`、`DBC`）。
如果 `format` 为 `SOURCE`，则必须指定 `language`。  `content` 参数包含 base64 编码的笔记本内容。 可以启用 `overwrite` 来覆盖现有笔记本。

```bash
curl -n -X POST -H 'Content-Type: application/json' -d \
'{
  "path": "/Users/user@example.com/new-notebook",
  "format": "SOURCE",
  "language": "SCALA",
  "content": "Ly8gRGF0YWJyaWNrcyBub3RlYm9vayBzb3VyY2UKcHJpbnQoImhlbGxvLCB3b3JsZCIpCgovLyBDT01NQU5EIC0tLS0tLS0tLS0KCg==",
  "overwrite": "false"
}' https://<databricks-instance>/api/2.0/workspace/import
```

如果请求成功，则返回空 JSON 字符串。

或者，可以通过多部分窗体发布导入笔记本。

```bash
curl -n -X POST https://<databricks-instance>/api/2.0/workspace/import \
       -F path="/Users/user@example.com/new-notebook" -F format=SOURCE -F language=SCALA -F overwrite=true -F content=@notebook.scala
```